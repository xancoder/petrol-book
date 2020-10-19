<template>
  <v-app>
    <v-app-bar app>
      <v-toolbar-title>{{ title }}</v-toolbar-title>
    </v-app-bar>

    <v-main>
      <v-container fluid>
        <router-view></router-view>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
const {label} = require('../../package')

export default {
  name: 'app',
  components: {},
  data: () => ({
    title: label
  })
}
</script>
