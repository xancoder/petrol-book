<template>
  <v-app>
    <v-app-bar app>
      <v-toolbar-title>{{ title }}</v-toolbar-title>
    </v-app-bar>

    <v-main>
      <router-view></router-view>
    </v-main>
  </v-app>
</template>

<script>
const {label} = require('../../package')

export default {
  components: {},
  name: 'app',
  data: () => ({
    title: label
  })
}
</script>
