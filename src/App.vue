<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script>
import axios from 'axios'
import Cookies from 'js-cookie'

export default {
  name: 'App',
  data() {},
  methods: {
    logMsg() {
      console.log(axios)
      console.log(Cookies)
    }
  }
}
</script>
