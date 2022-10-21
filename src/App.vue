<template>
  <header-bar
      :loggedIn="loggedIn"
      :username="username"
      :profile_img="profile_img_url"
      @login_status_change="changeLoginStatus"
  />
  <p v-if="!loggedIn">Please, login with your Spotify account</p>
  <main-window
      v-else
  />
</template>

<script>
import HeaderBar from "/@/components/HeaderBar.vue";
import SpotifyWebApi from 'spotify-web-api-js';
import MainWindow from "/@/components/MainWindow.vue";

export default {
  components: {MainWindow, HeaderBar},
  data() {
    return {
      loggedIn: localStorage.getItem('login_status') === 'logged_in',
      access_token: localStorage.getItem('access_token'),
      username: localStorage.getItem('display_name'),
      profile_img_url: localStorage.getItem('profile_img_url'),
    }
  },
  methods: {
    changeLoginStatus({loggedIn, access_token}) {
      this.loggedIn = loggedIn;
      this.access_token = access_token;
    },
    fetchUserData(accessToken) {
      const spotifyApi = new SpotifyWebApi();
      spotifyApi.setAccessToken(accessToken);
      spotifyApi.getMe({},(err,data) => {
        if (err) console.log(err);
        else {
          localStorage.setItem('display_name', data.display_name);
          localStorage.setItem('profile_img_url', data.images[0].url);
          this.username = data.display_name;
          this.profile_img_url = data.images[0].url;
        }
      });
      console.log(spotifyApi.getUserPlaylists())
    }


  },
  created() {
    if (this.loggedIn && !localStorage.getItem('display_name')) {
      this.fetchUserData(this.access_token);
    }
  }

}
</script>

<style>

</style>