<template>
  <div class="mx-3 mt-2 mb-3 about">
    <h1>О приложении</h1>
    Простое приложение для тестирования Vue в целом и Vuetify в частности.
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'About',
  data: () => ({
    dialog: false
  })
})
</script>
