<template>
  <header-bar
      :loggedIn="loggedIn"
      @login_status_change="changeLoginStatus"
  />

</template>

<script>
import HeaderBar from "/@/components/HeaderBar.vue";
export default {
  components: {HeaderBar},
  data() {
    return {
      loggedIn: localStorage.getItem('login_status') === 'logged_in',
      access_token: localStorage.getItem('access_token')
    }
  },
  methods: {
    changeLoginStatus({loggedIn,access_token}){
      this.loggedIn = loggedIn;
      this.access_token = access_token;
    }


  }

}
</script>

<style>

</style>