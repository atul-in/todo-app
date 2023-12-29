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
      <v-btn v-if="isUserLoggedIn" class="m-5 mr-5 blue--text" to="/tasks">Tasks</v-btn>
      <v-btn v-if="isUserLoggedIn && this.userRole != 'admin' && this.userRole != null" class="m-5 mr-5 yellow--text"
        to="/clients-data">Clients</v-btn>
      <v-btn v-if="isUserLoggedIn" class="m-5 red--text" @click="logout">Logout</v-btn>
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
    async logout() {
      try {
        await this.$auth.logout()
        this.$router.push('/login')
        this.snackbarText = "You are logged out successfully."
        this.snackbar = true
      } catch (error) {
        console.log(error)
      }
    },
  }
}
</script>
