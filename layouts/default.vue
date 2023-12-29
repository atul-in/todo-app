<template>
  <v-app dark>
    <v-app-bar :clipped-left="clipped" fixed app>

      <v-toolbar-title>
        <router-link to="/" class="white--text no-underline">
          <v-icon class="mr-2 mb-1">mdi-home-variant</v-icon>
          {{ appTitle }}
        </router-link>
      </v-toolbar-title>
      <v-spacer />
      <v-btn v-if="isUserLoggedIn" class="m-5 mr-5 blue--text" to="/tasks">My Tasks</v-btn>
      <!-- <v-btn v-if="isUserLoggedIn" class="m-5 mr-5 blue" @click.native.stop="openImportDialog">+ Import Todos</v-btn>
      <v-btn v-if="isUserLoggedIn" class="m-5 mr-5 green" @click.native.stop="downloadTasks">Download Todos</v-btn> -->
      <!-- <v-btn v-if="this.userRole != 'admin' && this.userRole != null" class="m-5 mr-5 blue--text" @click.native.stop="openTodoDialog">+ Add Todo</v-btn> -->
      <v-btn v-if="isUserLoggedIn" class="m-5 red--text" @click.native.stop="logoutUser">Logout</v-btn>
    </v-app-bar>
    <v-main>
      <v-container>
        <Nuxt />
        <client-only>
          <vue-confirm-dialog />
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

    logoutUser() {
      eventBus.$emit('logout-user');
    },

    //   openTodoDialog() {
    //     eventBus.$emit("open-todo-dialog");
    //   },

    //   openImportDialog() {
    //     eventBus.$emit("open-import-dialog");
    //   },

    //   downloadTasks() {
    //     eventBus.$emit('download-tasks');
    //   },
  }
}
</script>
