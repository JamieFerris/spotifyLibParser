<template>
  <div class="header">
    <h1>spotifyLibParser</h1>
    <div v-if="loggedIn">
      <div class="nav_wrapper">
        <img :src="profile_img" class="avatar">
        <div>
          <div>Hello, {{ username }}</div>
          <button @click="handleLogout" class="my_btn">logout</button>
        </div>
      </div>

      
    </div>
    <div v-else>
      <button @click="handleLogin">login</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "header-bar",
  props: {
    loggedIn: {
      type: Boolean,
      required: true
    },
    username: {
      type: String
    },
    profile_img: {
      type: String
    }
  },
  methods: {
    handleLogin() {
      const CLIENT_ID = '48dc52ec43a54e1fbe1b6073e88e4214';
      const REDIRECT_URI = 'http://127.0.0.1:5173/';
      const SPOTIFY_AUTHORIZE_ENDPOINT = 'https://accounts.spotify.com/authorize';
      const SCOPES = ['user-library-read', 'playlist-read-private', 'playlist-read-collaborative'];
      const SCOPES_PARAM = SCOPES.join('%20');

      //generating recommended state parameter:
      let state = new Uint8Array(8);
      crypto.getRandomValues(state);
      state = Array.from(state, (dec) => {
        return dec.toString(16).padStart(2, '0');
      }).join('');
      localStorage.setItem('state_param', state.toString());
      localStorage.setItem('login_status', 'redirected')
      window.location = `${SPOTIFY_AUTHORIZE_ENDPOINT}?client_id=${CLIENT_ID}&response_type=token&redirect_uri=${REDIRECT_URI}&state=${state}&scope=${SCOPES_PARAM}&show_dialog=true`;
    },
    getParamsFromAuth(href) {
      const paramsString = href.split('#')[1];
      return paramsString.split('&').reduce((result, currentValue) => {
        const [key, value] = currentValue.split('=');
        result[key] = value;
        return result;
      }, {});
    },
    handleLogout() {
      localStorage.clear()
      this.$emit('login_status_change', {loggedIn:false, access_token: ''})
    },
    handleLoginStatus() {
      if (localStorage.getItem('login_status') === 'redirected') {
        const params = this.getParamsFromAuth(window.location.href);
        const generated_state = localStorage.getItem('state_param')
        if (params.state === generated_state) {
          localStorage.setItem('access_token', params.access_token);
          localStorage.setItem('login_status', 'logged_in');
          localStorage.setItem('token_expires_at', (Date.now() + parseInt(params.expires_in)*1000).toString())
          this.$emit('login_status_change', {loggedIn:true, access_token: params.code})
        }
        window.location.href = window.location.origin;
      } else {
        if (parseInt(localStorage.getItem('token_expires_at')) < Date.now()) this.handleLogout();
      }
    },

  },
  //handling login status
  created() {
    this.handleLoginStatus();
  }
}
</script>

<style scoped>
  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .avatar {
    border-radius: 50%;
    height: 50px;
    width: 50px;
  }
  .my_btn {
    width: 100%;
  }
  .nav_wrapper {
    display: flex;
    flex-direction: row;
    align-items: center;
  }
</style>