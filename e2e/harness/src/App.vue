<template>
  <div id="app">
    <router-link to="/" tag="button" id='home-button'> Home </router-link>
    <button v-if='authState.isAuthenticated' v-on:click='logout' id='logout-button'> Logout </button>
    <button v-else v-on:click='login' id='login-button'> Login </button>
    <router-link to="/protected" tag="button"> Protected </router-link>
    <router-view/>
  </div>
</template>

<script>

export default {
  name: 'app',
  methods: {
    login () {
      this.$auth.signInWithRedirect()
    },
    async logout () {
      await this.$auth.signOut()
    }
  }
}
</script>
