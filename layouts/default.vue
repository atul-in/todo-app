<template>
  <v-app dark>
    <v-app-bar :clipped-left="clipped" fixed app>
      <!-- <v-app-bar-nav-icon @click.stop="drawer = !drawer" /> -->

        <v-toolbar-title v-if="!this.$auth">{{ title }}</v-toolbar-title>

        <v-toolbar-title v-else>Hello, {{ this.$auth.user.data.name }}</v-toolbar-title>

      <v-spacer />

      <v-btn class="m-5 mr-5 blue--text" @click.native.stop="openModal">+ New Todo</v-btn>
      <v-btn class="m-5 red--text" @click.native.stop="logoutUser">Logout</v-btn>


    </v-app-bar>
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import { eventBus } from "@/eventBus";

export default {
  name: 'DefaultLayout',
  data() {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      items: [
        {
          icon: 'mdi-apps',
          title: 'Welcome',
          to: '/'
        },
        {
          icon: 'mdi-chart-bubble',
          title: 'Inspire',
          to: '/inspire'
        }
      ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'TODO App'
    }
  },

  methods: {
    openModal() {
      eventBus.$emit("open-todo-modal");
    },

    logoutUser(){
      eventBus.$emit('logout-user');
    }
  }
}
</script>
