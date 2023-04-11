<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script>
import axios from 'axios'
import Cookies from 'js-cookie'
import { provide } from 'vue'

export default {
  name: 'App',
  data() {},
  methods: {
    logMsg() {
      console.log(axios)
      console.log(Cookies)
    }
  },
  setup() {
    provide('App', 'vue3')
  }
}
</script>
