<template>
  <v-app dark>
    <v-app-bar :clipped-left="clipped" fixed app>
      <v-toolbar-title>{{ appTitle }}</v-toolbar-title>
      <v-spacer />
      <v-btn v-if="this.userRole != 'admin' && this.userRole != null" class="m-5 mr-5 blue--text" @click.native.stop="openModal">+ Add Todo</v-btn>
      <v-btn v-if="isUserLoggedIn" class="m-5 red--text" @click.native.stop="logoutUser">Logout</v-btn>
    </v-app-bar>
    <v-main>
      <v-container>
        <Nuxt />
        <client-only>
          <vue-confirm-dialog/>
        </client-only>
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
      userRole: null,
    }
  },

  created() {
    if (this.$auth && this.$auth.loggedIn) {
      this.isUserLoggedIn = true;
      this.appTitle = "Hello, " + this.$auth.user.data.name;
      this.userRole = this.$auth.user.role[0];
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
