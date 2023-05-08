<template>
  <v-app>
    <v-app-bar
      app
      dense
      color="primary"
      dark
    >
    <v-toolbar-items class="hidden-sm-and-down">
        <v-btn color="grey accent-4" v-for="item in menu" :key="item.title" :to="item.link">{{
          item.title
        }}</v-btn>
      </v-toolbar-items>
    </v-app-bar>

    <v-main>
      <router-view/>
    </v-main>
  </v-app>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'App',

  data: () => ({
    menu: [
      { title: 'Generator', link: '/' }
    ]
  })
})
</script>
