<template>
  <div id="app">
    <h1>Witaj w systemie do zapisów na zajęcia</h1>

    <div v-if="authenticatedUsername">
      <UserPanel :username="authenticatedUsername" @logout="logMeOut()"></UserPanel>
      <MeetingsPage :username="authenticatedUsername"></MeetingsPage>
    </div>



    <div v-else>
      <button :class="signingUp ? 'button-outline' : ''" @click="signingUp = false">Logowanie</button>
      <button :class="!signingUp ? 'button-outline' : ''" @click="signingUp = true">Rejestracja</button>


      <div v-if="message" :class="accountCreated ? 'alert' : 'alert-red'"> {{message}}</div>

      <LoginForm v-if="!signingUp" @login="(user) => logMeIn(user)"></LoginForm>
      <LoginForm v-else @login="(user) => register(user)" button-label="załóż konto"></LoginForm>
    </div>
  </div>
</template>

<script>
import "milligram";
import LoginForm from "./LoginForm";
import UserPanel from "./UserPanel";
import MeetingsPage from "./meetings/MeetingsPage";
import axios from 'axios';

export default {
  components: {LoginForm, MeetingsPage, UserPanel},
  data() {
    return {
      message: '',
      accountCreated: false,
      signingUp: false,
      authenticatedUsername: '',
    }
  },
  mounted() {
        const username = localStorage.getItem('username');
        const token = localStorage.getItem('token');
        if (username && token) {
            this.authentication(username, token);
            axios.get(`/api/meetings`).catch(() => this.logMeOut());
        }
    },
  methods: {
    logMeIn(user) {
      axios.post('/api/tokens', user)
          .then(response => {
            const token = response.data.token;
            this.authentication(user.login, token);
          })
          .catch(response => {
            this.message = "Nie udało sie zalogować"
          });
    },
    logMeOut() {
      this.authenticatedUsername = '';
      delete axios.defaults.headers.common.Authorization;
      localStorage.clear();
    },
    register(user) {
      axios.post('/api/participants', user)
          .then(response => {
            this.message = "Udało się założyć konto"
            this.accountCreated = true
          })
          .catch(response => {
            this.message = "Nie udało się założyć konta"
            this.accountCreated = false
          });
    },
    authentication(username, token) {
          this.authenticatedUsername = username;
          axios.defaults.headers.common['Authorization'] = 'Bearer ' + token;
          localStorage.setItem('username', username);
          localStorage.setItem('token', token);
      },
  }
}
</script>

<style>
#app {
  max-width: 1000px;
  margin: 0 auto;
}

.alert {
  padding: 5px;
  border: 2px solid green;
  background: lightgreen;
  font-size: 2em;
  color: darkgreen;
  text-align: center;
}

.alert-red {
  padding: 5px;
  font-size: 2em;
  color: darkred;
  border: 2px solid red;
  background: lightcoral;
  text-align: center;
}
</style>
