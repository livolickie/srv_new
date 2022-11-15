<template>
  <v-container fill-height>
    <v-row>
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
                Клиентов: {{ clients.length }}
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

        <v-card class="pa-2 mt-4">
          <h3 class="text-center mb-2">Система: <span class="pa-1 success">OK</span></h3>
          <v-btn class="mb-2" color="">Перезагрузить</v-btn>
          <v-btn class="mb-2" color="">Отключить</v-btn>
          <v-btn class="mb-2" color="">Обновить</v-btn>
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
                <v-col lg="1">Лимит ({{maxPower}}):</v-col>
                <v-col>
                  <v-slider v-model="maxPower" step="10" thumb-label ticks></v-slider>
                </v-col>
              </v-row>
            </v-container>
          </template>
          <template v-if="tab == 1">
            <v-scroll-x-transition>
              <h1 class="text-center">Детали энергии</h1>
            </v-scroll-x-transition>
            <v-container>
              <v-row>
              <v-col lg="1">Лимит ({{maxEnergy}}):</v-col>
              <v-col>
                <v-slider v-model="maxEnergy" step="10" thumb-label ticks></v-slider>
              </v-col>
            </v-row>
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
  </v-container>
</template>

<script>

export default {
  name: 'IndexPage',
  data() {
    return {
      power: 0,
      energy: 0,
      clients: [
        {
          number: '8-800-555-35-35',
          speed: 12.5
        },
        {
          number: '8-900-555-35-35',
          speed: 15.5
        },
        {
          number: '8-700-555-35-35',
          speed: 9.5
        }
      ],

      maxPower: 10,
      maxEnergy: 10,

      network: 0,
      tab: 0,
      tHeaders: [
        { text: '№', value: 'id' },
        { text: 'Номер', value: 'number' },
        { text: 'Скорость', value: 'speed' },
      ],
      chartData: {
        labels: [],
        datasets: [
          {
            label: 'Сеть',
            data: [],
            fill: false,
            borderColor: 'rgb(75, 192, 192)'
          }
        ]
      },
      chartOptions: { responsive: true, maintainAspectRatio: false },
    }
  },
  computed: {
    tClients() {
      const clients = []
      let id = 0
      for (const client of this.clients) {
        clients.push({
          id,
          number: client.number,
          speed: client.speed
        })
        id++
      }
      return clients
    }
  },
  mounted() {
    setInterval(this.update, 1000)
  },
  methods: {
    selectTab(_tab) {
      this.tab = _tab
    },
    update() {
      this.power++
      this.energy++
      this.network++

      this.chartData.labels.push(new Date().toLocaleTimeString())
      this.chartData.datasets[0].data.push(this.network)
    }
  },
}
</script>
