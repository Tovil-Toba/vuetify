<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          style="cursor: pointer"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
          @click="$router.push({ name: 'home' }).catch(err => {})"
        />

        <v-img
          alt="Vuetify Name"
          class="shrink mt-1 mr-3 hidden-sm-and-down"
          style="cursor: pointer"
          contain
          min-width="100"
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-name-dark.png"
          width="100"
          @click="$router.push({ name: 'home' }).catch(err => {})"
        />
      </div>

      <v-btn
        icon
        @click="$router.push({ name: 'map' }).catch(err => {})"
      >
        <v-icon>mdi-airplane</v-icon>
      </v-btn>

      <v-btn
        icon
        @click="$router.push({ name: 'currencies' })"
      >
        <v-icon>mdi-currency-usd</v-icon>
      </v-btn>

      <v-btn
        icon
        @click="$router.push({ name: 'about' })"
      >
        <v-icon>mdi-information</v-icon>
      </v-btn>

      <v-spacer />

      <v-btn
        icon
        @click="toggleTheme"
      >
        <v-icon>{{ toggleThemeIcon }}</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <router-view />
    </v-main>
  </v-app>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'App',

  components: {
    //
  },

  data: () => ({
    //
  }),

  computed: {
    toggleThemeIcon: function () {
      return this.$vuetify.theme.dark
        ? 'mdi-weather-night' // moon-waxing-crescent
        : 'mdi-white-balance-sunny'
    }
  },

  methods: {
    toggleTheme () {
      this.$vuetify.theme.dark = !this.$vuetify.theme.dark
    }
  }
})
</script>
