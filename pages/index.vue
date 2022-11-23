<!-- eslint-disable vue/valid-v-slot -->
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

            <v-list-item @click="selectTab(4)">
              <v-list-item-icon>
                <v-icon>mdi-map</v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                Карта сети
              </v-list-item-content>
            </v-list-item>
          </v-list-item-group>
        </v-list>

        <v-card class="pa-2 mt-4 d-flex flex-column">
          <h3 class="text-center mb-2">Система: <span class="pa-1" :class="{
            'success': !startError, 'warning': startError
          }">
              {{ !startError ? 'ОК' : 'АВАРИЯ' }}
            </span></h3>
          <v-btn class="mb-2" @click="status = 'reload'; reload()">Перезагрузить</v-btn>
          <v-btn v-if="status != 'reload' && !startError" class="mb-2" :color="status != 'pause' ? '' : 'success'"
            @click="status == 'active' ? status = 'pause' : status = 'active'; snack = status == 'pause'; snackMessage = 'Система поставлена на паузу'">
            {{ status != 'active' ? 'Возобновить' :
                'Пауза'
            }}</v-btn>
          <v-btn v-if="startError" @click="handleError()">Автоматика</v-btn>
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
                <v-col lg="1">Мощность ({{ power }} dbW):</v-col>
                <v-col>
                  <v-slider v-model="power" min="50" max="180" step="10" thumb-label ticks></v-slider>
                </v-col>
              </v-row>
              <!-- <h3>Затрачено: {{ spentPower }} dbW</h3> -->
            </v-container>
          </template>
          <template v-if="tab == 1">
            <v-scroll-x-transition>
              <h1 class="text-center">Детали энергии</h1>
            </v-scroll-x-transition>
            <v-container>
              <!-- <v-row> -->
              <!-- <v-col lg="1">Лимит ({{ maxEnergy }}W):</v-col>
                <v-col>
                  <v-slider v-model="maxEnergy" min="350" max="800" step="10" thumb-label ticks></v-slider>
                </v-col>
              </v-row>-->
              <h3 class="text-center">Затрачено: {{ spentEnergy.toFixed(1) }}W</h3>
            </v-container>
          </template>
          <template v-if="tab == 2">
            <v-scroll-x-transition>
              <h1 class="text-center">Клиенты</h1>
            </v-scroll-x-transition>
            <v-data-table :headers="tHeaders" :items="tClients" :items-per-page="5" class="elevation-1">
              // eslint-disable-next-line vue/valid-v-slot, vue/valid-v-slot
              <template v-slot:item.actions="{ item }">
                <v-btn @click="item.obj.active = false">Отключить</v-btn>
              </template>
            </v-data-table>
          </template>
          <template v-if="tab == 3">
            <v-scroll-x-transition>
              <h1 class="text-center">Детали сети</h1>
            </v-scroll-x-transition>
            <v-container>
              <v-row>
                <!-- <v-col lg="1">Лимит ({{ maxNetwork }} MB/s):</v-col> -->
                <!-- <v-col>
                  <v-slider v-model="maxNetwork" min="800" max="1500" step="25" thumb-label ticks></v-slider>
                </v-col> -->
              </v-row>
            </v-container>

            <line-chart :chart-options='chartOptions' :chart-data='chartData' chart-id='myCustomId' />
          </template>
          <template v-if="tab == 4">
            <v-scroll-x-transition>
              <h1 class="text-center">Карта сети</h1>
            </v-scroll-x-transition>
            <v-container>
              <canvas id="canvas" width="800" height="800" class="mt-5" style="margin: 0 auto"></canvas>
            </v-container>
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


    <v-snackbar v-model="snack" :multi-line="multiLine">
      {{ snackMessage }}
    </v-snackbar>

    <v-snackbar v-model="errorSnack" :timeout="2000">
      {{ errorMessage }}
      <template #action="{ attrs }">
        <v-btn color="blue" text v-bind="attrs" @click="snackbar = false">
          Закрыть
        </v-btn>
      </template>
    </v-snackbar>

  </v-container>
</template>

<script>

/*
  power - manual
  energy = power * 3.5 + random()
  heat = energy * 10 || Red Signal on HEAT > 75%
  network = each_client.dist_to_station * (AREA_TYPE) + random() || limit (100MB - 2000MB)
  || RED SIGNAL ON WHERE 75% OF CLIENTS SPEED < 25%
*/

export default {
  name: 'IndexPage',
  data() {
    return {
      status: 'active',
      power: 100,
      energy: 0,
      clients: [
      ],
      load: 0,
      loadInterval: null,

      spentPower: 0,
      spentEnergy: 0,
      errorMessage: '',

      maxPower: 120,
      maxEnergy: 420,
      maxNetwork: 1000,
      startError: false,
      errorSnack: false,

      snackMessage: '',
      snack: false,

      network: 0,
      tab: 0,
      tHeaders: [
        { text: '№', value: 'id' },
        { text: 'Номер', value: 'number' },
        { text: 'Скорость', value: 'speed' },
        { text: 'Действия', value: 'actions' },
      ],
      canvas: null,
      chartData: {
        labels: [],
        datasets: [
          {
            label: 'Сеть',
            data: [],
            fill: true,
            borderColor: 'rgb(75, 192, 192)'
          },
          // {
          //   label: 'Мощность',
          //   data: [],
          //   fill: true,
          //   borderColor: 'rgb(155, 48, 217)'
          // },
          // {
          //   label: 'Электроэнергия',
          //   data: [],
          //   fill: true,
          //   borderColor: 'rgb(77, 219, 37)'
          // }
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
          y: client.y,
          obj: client
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
    init() {
      this.startError = false
      this.chartData = {
        labels: [],
        datasets: [
          {
            label: 'Сеть',
            data: [],
            fill: true,
            borderColor: 'rgb(75, 192, 192)'
          },
          // {
          //   label: 'Мощность',
          //   data: [],
          //   fill: true,
          //   borderColor: 'rgb(155, 48, 217)'
          // },
          // {
          //   label: 'Электроэнергия',
          //   data: [],
          //   fill: true,
          //   borderColor: 'rgb(77, 219, 37)'
          // }
        ]
      }
      this.clients = []
      for (let i = 0; i < 2000; i++) {
        this.clients.push({
          number: `8-700-${Math.round(Math.random() * 200 + 300)}-${Math.round(Math.random() * 55 + 20)}-${Math.round(Math.random() * 35 + 10)}`,
          speed: parseFloat((Math.random() * 10 + 5).toFixed(1)),
          active: Math.random() > 0.773,
          x: Math.random() * 800,
          y: Math.random() * 800
        })

        this.status = 'active'
      }
    },
    reload() {
      clearInterval(this.loadInterval)
      this.power = 100 + Math.random() * 10
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
      if (_tab === 4) {
        setTimeout(() => {
          this.canvas = document.querySelector('#canvas')
        }, 250)
      }
      this.tab = _tab
    },
    handleError() {
      this.power = 100 + 10 * Math.random()
      this.startError = false;
    },
    errorHandler(reason = '') {
      this.errorMessage = reason
      this.errorSnack = true
      this.status = 'pause'
    },
    update() {

      if (this.status !== 'active') return

      if (!this.startError) {
        if (this.power < 60) {
          this.startError = true
          setTimeout(() => this.errorHandler('Мощность антенны слишком мала! Система поставлена паузу', 1500 + Math.random() * 1000))
        }

        if (this.power > 160) {
          this.startError = true
          setTimeout(() => this.errorHandler('Мощность антенны слишком высока! Система поставлена паузу', 1500 + Math.random() * 1000))
        }
      }

      this.network = 0
      // Сеть будет зависеть от клиентов
      for (const client of this.clients) {

        if (Math.random() < 0.95) {
          if (client.x < 800 && Math.random() > 0.55) client.x += Math.random() * 5
          else client.x -= Math.random() * 2.5
        }
        else if (Math.random() > 9.5) {
          if (client.y < 800 && Math.random() < 0.55) client.y += Math.random() * 5
          else client.y -= Math.random() * 2.5
        }

        client.distance = Math.sqrt((Math.pow(client.x - 392, 2) + Math.pow(client.y - 392, 2)))
        client.speed = client.distance * 0.01 + this.power / 50
        if (client.active && Math.random() > 0.995) {
          client.active = false
        }
        else if (!client.active && Math.random() < 0.005) client.active = true

        if (client.active) {
          const rnd = Math.random()
          if (rnd > 0.98) {
            const changeVal = Math.random() * 1 + 0.5
            if (client.speed < 15) client.speed += changeVal
            else client.speed -= changeVal

            client.speed = parseFloat(client.speed.toFixed(1))
          }

          this.network += client.speed

          client.speed = Number.parseFloat(client.speed.toFixed(1))
        }
      }
      this.network = Math.round(this.network)


      this.spentPower += this.power
      this.spentEnergy += this.energy
      this.spentPower = parseFloat(this.spentPower.toFixed(1))
      this.spendEnergy = parseFloat(this.spentEnergy.toFixed(1))
      // Мощность зависит от сети
      // this.power = Math.round((this.network / 10 + Math.random()) + 5 * Math.random())

      this.energy = parseFloat((this.power * 4.5 + 3 * Math.random()).toFixed(1))

      // Генерация рандомных данных (отлючение - подключение клиентов)
      if (this.canvas) {
        const ctx = this.canvas.getContext('2d')
        ctx.clearRect(0, 0, 800, 800)
        ctx.fillStyle = 'rgb(18,18,18)'
        ctx.fillRect(0, 0, 800, 800)

        ctx.strokeStyle = 'rgb(68, 240, 34)'
        for (const client of this.clients) {

          ctx.fillStyle = 'rgb(68, 240, 34)'

          // Canvas
          if (client.active) {
            if (client.distance > this.power * 4.5) {
              ctx.fillStyle = 'rgb(255, 200, 200)'
            }
            ctx.beginPath()
            ctx.arc(client.x, client.y, 2, 0, 2 * Math.PI)
            ctx.fill()
          }
        }


        ctx.font = '12px Roboto'
        // 2G
        const r3 = this.power * 4.5
        ctx.globalAlpha = 0.1
        ctx.fillStyle = 'rgb(91, 97, 91)'
        ctx.beginPath()
        ctx.arc(392, 392, r3, 0, 2 * Math.PI)
        ctx.fill()

        // 3G
        const r2 = this.power * 3.25
        ctx.globalAlpha = 0.1
        ctx.fillStyle = 'rgb(73, 171, 72)'
        ctx.beginPath()
        ctx.arc(392, 392, r2, 0, 2 * Math.PI)
        ctx.fill()

        // Вычисление радиуса на основе мощность (4G сеть)
        const r = this.power * 1.5
        ctx.globalAlpha = 0.1
        ctx.fillStyle = 'rgb(255, 0, 251)'
        ctx.beginPath()
        ctx.arc(392, 392, r, 0, 2 * Math.PI)
        ctx.fill()

        ctx.globalAlpha = 1
        ctx.strokeStyle = 'rgb(255, 255, 255)'
        ctx.strokeRect(400 - 16, 400 - 16, 16, 16)

        ctx.fillStyle = 'rgb(255,255,255)'
        ctx.fillText('Станция', 405, 395)
      }

      if (this.chartData.datasets[0].data.length > 400) {
        this.chartData.datasets[0].data.shift()
        // this.chartData.datasets[1].data.shift()
        // this.chartData.datasets[2].data.shift()
      }
      if (this.chartData.labels.length > 600) {
        this.chartData.labels.shift()
      }
      this.chartData.labels.push(new Date().toLocaleTimeString())
      this.chartData.datasets[0].data.push(this.network)
      // this.chartData.datasets[1].data.push(this.power)
      // this.chartData.datasets[2].data.push(this.energy)
    }
  },
}
</script>
