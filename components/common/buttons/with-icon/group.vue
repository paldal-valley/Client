<template>
  <div>
    <v-layout row class="px-0 py-0 mb-2">
      <vue-button
        v-for="(btn, i) in buttons"
        :key="i"
        :text="btn.text"
        :to="btn.to"
      />
    </v-layout>
  </div>
</template>

<script>
import VueButton from './index'
export default {
  components: {
    VueButton
  },
  props: {
    buttons: {
      type: Array,
      default: () => []
    }
  }
}
</script>

<style scoped></style>
