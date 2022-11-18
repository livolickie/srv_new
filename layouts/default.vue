<template>
  <v-app dark>
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
    <v-footer :absolute="!fixed" app class="d-flex justify-space-between">
      <span>&copy;  - {{ new Date().getFullYear() }}</span>
      <div class="d-flex flex-row align-center">
        <v-switch v-model="$vuetify.theme.dark" ></v-switch>
        <span>Время: {{ time }}</span>
      </div>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  name: 'DefaultLayout',
  data() {
    return {
      time: ''
    }
  },
  mounted() {
    this.updateTime()
    setInterval(this.updateTime, 1000)
  },
  methods: {
    updateTime() {
      const date = new Date()
      this.time = `${date.getHours()}:${date.getMinutes()}:${date.getSeconds()}`
    }
  },
}
</script>
