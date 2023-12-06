<template>
  <v-app dark>
    <v-app-bar :clipped-left="clipped" fixed app>
      <v-toolbar-title>{{ appTitle }}</v-toolbar-title>
      <v-spacer />
      <v-btn v-if="this.role_id != 1 && this.role_id != null" class="m-5 mr-5 blue--text" @click.native.stop="openModal">+ Add Todo</v-btn>
      <v-btn v-if="isUserLoggedIn" class="m-5 red--text" @click.native.stop="logoutUser">Logout</v-btn>
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
      miniVariant: false,
      right: true,
      rightDrawer: false,
      isUserLoggedIn: false,
      appTitle: 'TODO App',
      role_id: null,
    }
  },

  created() {
    if (this.$auth && this.$auth.loggedIn) {
      this.isUserLoggedIn = true;
      this.appTitle = "Hello, " + this.$auth.user.data.name;
      this.role_id = this.$auth.user.data.role_id;
    }
  },

  methods: {

    openModal() {
      eventBus.$emit("open-todo-modal");
    },

    logoutUser() {
      eventBus.$emit('logout-user');
    },
  }
}
</script>
