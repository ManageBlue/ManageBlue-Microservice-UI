<template>
  <v-app dark>

    <v-navigation-drawer
      v-model="drawer"
      :mini-variant="miniVariant"
      clipped
      :permanent="$vuetify.breakpoint.mdAndUp"
      fixed
      app
    >
      <v-list>
        <v-list-item
          v-for="(item, i) in items"
          :key="i"
          :to="item.to"
          router
          exact
        >
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>


    <v-app-bar clipped-left fixed app>
      <v-app-bar-nav-icon class="hidden-md-and-up" @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-btn
        icon
        @click.stop="miniVariant = !miniVariant"
      >
        <v-icon>mdi-{{ `chevron-${miniVariant ? 'right' : 'left'}` }}</v-icon>
      </v-btn>
      <v-toolbar-title v-text="title" />
      <v-spacer />

      <!-- PROFILE view -->
      <v-tooltip bottom>
        <template v-slot:activator="{ on, attrs }">
          <v-btn nuxt icon v-bind="attrs" v-on="on" :to="({name: 'profile'})" exact>
            <v-icon >mdi-account</v-icon>
          </v-btn>
        </template>
        <span>Profile</span>
      </v-tooltip>

    </v-app-bar>

    <v-main>
      <v-container>
        <nuxt />
      </v-container>
    </v-main>

    <v-footer absolute app>
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
import {mapActions} from "vuex";

export default {
  data () {
    return {
      clipped: true,
      drawer: false,
      items: [
        {
          icon: 'mdi-view-dashboard',
          title: 'Dashboard',
          to: '/'
        },
        {
          icon: 'mdi-archive',
          title: 'Archive',
          to: '/archive'
        }
      ],
      miniVariant: false,
      right: true,
      title: 'ManageBLUE'
    }
  },
  methods:{
    /*...mapActions({
      logoutAction: 'auth/logout'
    }),*/

    async logout() {
      await this.$auth.logout();
      // this.$auth.$storage.removeUniversal('user')
    },
  },
  // TODO: replace with isLoggedIn
  /*mounted() {
    this.$axios.get('auth/verify')
      .catch(error => {
        // logout if api call unAuthorized
        if (error.response && error.response.status === 401)
          this.globalUnAuthorization()
      })
  }*/
}
</script>
