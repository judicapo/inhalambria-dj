<template>
  <div id="app">
    <div id="nav">
      <template>
        <b-navbar>
          <template slot="brand">
            <b-navbar-item tag="router-link" :to="{ path: '/' }">
              <i class="fas fa-3x fa-expand-arrows-alt"></i> inhalambria DJ
            </b-navbar-item>
          </template>
          <template slot="start">
            <!-- <b-navbar-dropdown label="Info">
              <b-navbar-item href="#">
                  About
              </b-navbar-item>
              <b-navbar-item href="#">
                  Contact
              </b-navbar-item>
            </b-navbar-dropdown> -->

            <b-navbar-item
              v-if="user()"
              tag="router-link"
              :to="{ path: '/favorites' }"
            >
              Favorites
            </b-navbar-item>
            <b-navbar-item tag="router-link" :to="{ path: '/about' }">
              About
            </b-navbar-item>
          </template>

          <template slot="end">
            <b-navbar-item v-if="!user()" tag="div">
              <div class="buttons">
                <router-link
                  tag="a"
                  class="button is-primary"
                  :to="{ path: '/register' }"
                >
                  <strong>Sign up</strong>
                </router-link>
                <router-link
                  tag="a"
                  class="button is-light"
                  :to="{ path: '/login' }"
                >
                  Log in
                </router-link>
              </div>
            </b-navbar-item>
            <b-navbar-item v-else tag="div">
              <div class="buttons" v-on:click="logoutApp">
                <router-link
                  tag="a"
                  class="button is-light"
                  :to="{ path: '/' }"
                >
                  Log out
                </router-link>
              </div>
            </b-navbar-item>
          </template>
        </b-navbar>
      </template>
    </div>
    <router-view />
    <footer class="footer">
      <div class="content has-text-centered">
        <p>
          <strong
            >Made with 💖 and ☕ by
            <a href="https://www.github.com/judicapo" target="_blank">
              <i class="fab fa-github"> judicapo</i></a
            ></strong
          >
        </p>
        <p>
          This site uses Vue Js, Buefy and Font Awesome technologies to improve
          your experience.
        </p>
      </div>
    </footer>
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";

export default {
  name: "App",
  data() {
    return {
      authUser: undefined
    };
  },
  computed: {},
  methods: {
    ...mapGetters("auth", ["isAuthenticated", "user"]),
    ...mapActions("auth", ["logout", "authenticate"]),
    logoutApp: function() {
      this.logout().then(() => (this.authUser = undefined));
    }
  },
  watch: {},
  updated() {},
  created() {},
  mounted() {
    this.authenticate()
      .then(user => {
        this.authUser = user;
      })
      .catch(() => {
        this.$router.push("/login");
      });
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
}
</style>
