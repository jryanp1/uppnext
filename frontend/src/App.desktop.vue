<template>
  <div>


    <link href='https://fonts.googleapis.com/css?family=Roboto:300,400,500,700|Material+Icons' rel="stylesheet">
    <router-view ></router-view>



  </div>
</template>

<script>
import * as socketIoClient from 'socket.io-client';
import * as sailsIo from 'sails.io.js';
import { mapMutations } from 'vuex';
import AppMixin from './App.mixin';

const HelpIndex = () => import('./components/help/Help.desktop');
const Landing = () => import('./components/Landing.desktop');
export default {
  mixins: [AppMixin],

  components: {
    HelpIndex,
    Landing
  },

  data: () => ({
    io: null
  }),

  computed: {
    isHelpVisible: {
      get() {
        return this.$store.state.isHelpVisible;
      },

      set(isHelpVisible) {
        this.store.commit('SET_IS_HELP_VISIBLE', isHelpVisible);
      }
    }
  },

  watch: {
    isHelpVisible() {
      if (!this.isHelpVisible) this.io.socket.disconnect();
      if (this.isHelpVisible && this.io) this.io.socket.reconnect();
    }
  },

  methods: {
    setIo() {
      if (!this.io) {
        let io = sailsIo(socketIoClient);

        let isProductionEnvironment = process.env.NODE_ENV === 'production';
        let url;

        if (isProductionEnvironment) {
          url = `${location.protocol}//${location.hostname}${
            location.port ? ':' + location.port : ''
          }`;
        } else url = 'http://localhost:1337';

        io.sails.url = url;
        io.sails.environment = process.env.NODE_ENV || 'development';
        io.sails.useCORSRouteToGetCookie = false;

        this.$set(this, 'io', io);
      }
    },

    logout() {
      this.deleteCookie('user');
      this.isUserAuthenticated = false;
      localStorage.clear();

      this.$router.push({ name: 'Home' });
    },

    ...mapMutations({
      setIsHelpVisible: 'SET_IS_HELP_VISIBLE'
    })
  }
};
</script>
