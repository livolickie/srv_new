<template>
  <v-container fill-height>
    <v-row v-if="status != 'reload'">
      <v-col lg="2">
        <v-list dense>
          <v-subheader>Показатели</v-subheader>
          <v-list-item-group>
            <v-list-item @click="selectTab(0)">
              <v-list-item-icon>
                <v-icon>mdi-meter-electric</v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                Мощность: {{ power }} dbW
              </v-list-item-content>
            </v-list-item>

            <v-list-item @click="selectTab(1)">
              <v-list-item-icon>
                <v-icon>mdi-lightning-bolt-circle</v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                Энергия: {{ energy }}W
              </v-list-item-content>
            </v-list-item>

            <v-list-item @click="selectTab(2)">
              <v-list-item-icon>
                <v-icon>mdi-account-multiple</v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                Клиентов: {{ activeClients.length }}
              </v-list-item-content>
            </v-list-item>

            <v-list-item @click="selectTab(3)">
              <v-list-item-icon>
                <v-icon>mdi-wan</v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                Сеть: {{ network }} MB/s
              </v-list-item-content>
            </v-list-item>
          </v-list-item-group>
        </v-list>

        <v-card class="pa-2 mt-4 d-flex flex-column">
          <h3 class="text-center mb-2">Система: <span class="pa-1 success">OK</span></h3>
          <v-btn class="mb-2" @click="status = 'reload'; reload()">Перезагрузить</v-btn>
          <v-btn v-if="status != 'reload'" class="mb-2" :color="status != 'pause' ? '' : 'success'"
            @click="status == 'active' ? status = 'pause' : status = 'active'; snack = status == 'pause'; snackMessage='Система поставлена на паузу'">{{ status != 'active' ? 'Возобновить' :
                'Пауза'
            }}</v-btn>
        </v-card>
      </v-col>

      <v-col>
        <v-card class="q-">
          <template v-if="tab == 0">
            <v-scroll-x-transition>
              <h1 class="text-center">Детали мощности</h1>
            </v-scroll-x-transition>
            <v-container>
              <v-row>
                <v-col lg="1">Лимит ({{ maxPower }}):</v-col>
                <v-col>
                  <v-slider v-model="maxPower" min="90" max="180" step="10" thumb-label ticks></v-slider>
                </v-col>
              </v-row>
              <h3>Затрачено: {{spentPower}} dbW</h3>
            </v-container>
          </template>
          <template v-if="tab == 1">
            <v-scroll-x-transition>
              <h1 class="text-center">Детали энергии</h1>
            </v-scroll-x-transition>
            <v-container>
              <v-row>
                <v-col lg="1">Лимит ({{ maxEnergy }}):</v-col>
                <v-col>
                  <v-slider v-model="maxEnergy" min="350" max="800" step="10" thumb-label ticks></v-slider>
                </v-col>
              </v-row>
              <h3>Затрачено: {{spentEnergy}}W</h3>
            </v-container>
          </template>
          <template v-if="tab == 2">
            <v-scroll-x-transition>
              <h1 class="text-center">Клиенты</h1>
            </v-scroll-x-transition>
            <v-data-table :headers="tHeaders" :items="tClients" :items-per-page="5" class="elevation-1"></v-data-table>
          </template>
          <template v-if="tab == 3">
            <v-scroll-x-transition>
              <h1 class="text-center">Детали сети</h1>
            </v-scroll-x-transition>

            <line-chart :chart-options='chartOptions' :chart-data='chartData' chart-id='myCustomId' />
          </template>
        </v-card>
      </v-col>

    </v-row>
    <v-row v-else justify="center">

      <v-progress-circular :rotate="360" :size="300" :width="15" :value="load" color="teal" class="text-center"
        style="margin-top: 15%">
        <div class="d-flex flex-column">
          <span>
            Идёт перезагрузка
          </span>
          <span class="fw-bold">
            {{ load }}%
          </span>
        </div>
      </v-progress-circular>
    </v-row>

    <canvas width="500" height="400" id="canvas" class="mt-5"></canvas>

    <v-snackbar
      v-model="snack"
      :multi-line="multiLine"
    >
      {{ snackMessage }}
    </v-snackbar>

  </v-container>
</template>

<script>

export default {
  name: 'IndexPage',
  data() {
    return {
      status: 'active',
      power: 0,
      energy: 0,
      clients: [
      ],
      load: 0,
      loadInterval: null,

      spentPower: 0,
      spentEnergy: 0,
      
      maxPower: 120,
      maxEnergy: 420,

      snackMessage: '',
      snack: false,

      network: 0,
      tab: 0,
      tHeaders: [
        { text: '№', value: 'id' },
        { text: 'Номер', value: 'number' },
        { text: 'Скорость', value: 'speed' },
      ],
      ctx: null,
      chartData: {
        labels: [],
        datasets: [
          {
            label: 'Сеть',
            data: [],
            fill: true,
            borderColor: 'rgb(75, 192, 192)'
          },
          {
            label: 'Мощность',
            data: [],
            fill: true,
            borderColor: 'rgb(155, 48, 217)'
          },
          {
            label: 'Электроэнергия',
            data: [],
            fill: true,
            borderColor: 'rgb(77, 219, 37)'
          }
        ]
      },
      chartOptions: { responsive: true, maintainAspectRatio: false },
    }
  },
  computed: {
    activeClients() {
      const clients = this.clients.filter(client => client.active)
      return clients;
    },
    tClients() {
      const clients = []
      let id = 0
      for (const client of this.activeClients) {
        clients.push({
          id,
          number: client.number,
          speed: client.speed,
          x: client.x,
          y: client.y
        })
        id++
      }
      return clients
    }
  },
  mounted() {
    this.init()
    setInterval(this.update, 500)
  },
  methods: {
    canvas() {
      const canvas = document.querySelector('#canvas')
      this.ctx = canvas.getContext('2d')
      this.ctx.fillStyle = 'rgb(100, 100, 100)'
      this.ctx.fillRect(0, 0, 800, 600)
    },
    init() {
      this.chartData = {
        labels: [],
        datasets: [
          {
            label: 'Сеть',
            data: [],
            fill: true,
            borderColor: 'rgb(75, 192, 192)'
          },
          {
            label: 'Мощность',
            data: [],
            fill: true,
            borderColor: 'rgb(155, 48, 217)'
          },
          {
            label: 'Электроэнергия',
            data: [],
            fill: true,
            borderColor: 'rgb(77, 219, 37)'
          }
        ]
      }
      this.clients = []
      for (let i = 0; i < 2000; i++) {
        this.clients.push({
          number: `8-700-${Math.round(Math.random() * 200 + 300)}-${Math.round(Math.random() * 55 + 20)}-${Math.round(Math.random() * 35 + 10)}`,
          speed: parseFloat((Math.random() * 10 + 5).toFixed(1)),
          active: Math.random() > 0.573,
          x: Math.random() * 500,
          y: Math.random() * 400
        })

        this.status = 'active'
      }
      this.canvas()
    },
    reload() {
      clearInterval(this.loadInterval)
      this.load = 0
      this.loadInterval = setInterval(() => {
        if (this.reload) {
          this.load += 14
        } else {
          clearInterval(this.loadInterval)
        }
      }, 300)
      setTimeout(this.init, 3000)
    },
    selectTab(_tab) {
      this.tab = _tab
    },
    update() {

      if (this.status !== 'active') return

      // Генерация рандомных данных (отлючение - подключение клиентов)
      this.ctx.fillStyle = 'rgb(100, 100, 100)'
      this.ctx.fillRect(0, 0, 800, 600)
      this.ctx.fillStyle = 'rgb(0, 100, 0)'
      for (const client of this.clients) {
        if (client.active && Math.random() > 0.99) {
          client.active = false
        }
        else if (!client.active && Math.random() < 0.01) client.active = true

        if (client.active) {
          const rnd = Math.random()
          if (rnd > 0.98) {
            const changeVal = Math.random() * 1 + 0.5
            if (client.speed < 15) client.speed += changeVal
            else client.speed -= changeVal

            client.speed = parseFloat(client.speed.toFixed(1))
          }


          if (Math.random() < 0.25) {
            if (client.x < 500 && Math.random() > 0.5) client.x += Math.random() * 5
            else client.x -= Math.random() * 5
          }
          else if (Math.random() > 0.75) {
            if (client.y < 500 && Math.random() < 0.5) client.y += Math.random() * 5
            else client.y -= Math.random() * 5
          }

          // Canvas
          this.ctx.beginPath()
          this.ctx.arc(client.x, client.y, 2, 0, 2 *Math.PI)
          this.ctx.stroke()
        }
      }

      this.network = 0
      // Сеть будет зависеть от клиентов
      for (const client of this.activeClients) {
        this.network += client.speed / 10
      }
      this.network = Math.round(this.network)


      this.spentPower += this.power
      this.spentEnergy += this.energy
      this.spentPower = parseFloat(this.spentPower.toFixed(1))
      this.spendEnergy = parseFloat(this.spentEnergy.toFixed(1))
      // Мощность зависит от сети
      this.power = Math.round((this.network / 10 + Math.random()) + 5 * Math.random())

      this.energy = parseFloat((this.power * 3.5 + 3 * Math.random()).toFixed(1))

      if (this.chartData.datasets[0].data.length > 400) {
        this.chartData.datasets[0].data.shift()
        this.chartData.datasets[1].data.shift()
        this.chartData.datasets[2].data.shift()
      }
      if (this.chartData.labels.length > 600) {
        this.chartData.labels.shift()
      }
      this.chartData.labels.push(new Date().toLocaleTimeString())
      this.chartData.datasets[0].data.push(this.network)
      this.chartData.datasets[1].data.push(this.power)
      this.chartData.datasets[2].data.push(this.energy)
    }
  },
}
</script>
