<template>
  <div id="app">

    <Login v-if="!userIsLogged && !userIsLogIn"
      @userIsLogIn="userIsLogIn = !userIsLogIn" />

    <v-app v-else id="inspire">

      <v-navigation-drawer
        fixed
        v-model="drawer"
        app
      >
        <v-list dense>
<!--          <v-list-tile @click="">-->
<!--            <v-list-tile-action>-->
<!--              <v-icon>home</v-icon>-->
<!--            </v-list-tile-action>-->
<!--            <v-list-tile-content>-->
<!--              <v-list-tile-title>Home</v-list-tile-title>-->
<!--            </v-list-tile-content>-->
<!--          </v-list-tile>-->
          <v-list-tile>
            <v-list-tile-action>
              <v-icon>contact_mail</v-icon>
            </v-list-tile-action>
            <v-list-tile-content>
              <v-list-tile-title>Подписчики</v-list-tile-title>
            </v-list-tile-content>
          </v-list-tile>
        </v-list>
      </v-navigation-drawer>

      <v-toolbar color="blue-grey" dark fixed app>
        <v-toolbar-side-icon @click.stop="drawer = !drawer"></v-toolbar-side-icon>
        <v-toolbar-title>Админ панель Агровести</v-toolbar-title>
      </v-toolbar>

      <v-content>
        <v-container fluid fill-height>
          <v-layout
            justify-center
            align-top
          >
            <v-flex text-xs-center>

              <Subscribers slot="activator" />

            </v-flex>
          </v-layout>
        </v-container>
      </v-content>

      <v-footer color="blue-grey" app inset>
        <span class="white--text">&copy; 2019</span>
      </v-footer>
    </v-app>
  </div>
</template>

<script>
// const Login = () => import(`@/components/Login`);
// const Subscribers = () => import(`@/components/Subscribers`);
import Login from '@/components/Login'
import Subscribers from '@/components/Subscribers'

export default {
  name: 'App',
  components: {
    Subscribers,
    Login
  },
  data () {
    return {
      drawer: null,
      userIsLogIn: false
    }
  },
  computed: {
    userIsLogged () {
      return sessionStorage.getItem('agroAdminAuth') &&
        sessionStorage.getItem('agroAdminAuth') === 'isAuth'
    }
  }
}
</script>
