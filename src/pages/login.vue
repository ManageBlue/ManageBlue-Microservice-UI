<template>
  <div class="background">
    <v-container fill-height>
      <v-row justify="center">
        <v-card width="400" class="ma-3" elevation="12">
          <v-toolbar dark color="primary">
            <v-avatar size="50" class="my-1 mr-3" tile>
              <!--<v-img src="@/assets/tarca.png"/>-->
            </v-avatar>
            <h1>Login</h1>
          </v-toolbar>
          <v-card-text>
            <v-form ref="form" @submit.prevent="userLogin">
              <v-text-field
                v-model="form.username"
                label="Username"
                prepend-icon="mdi-account-circle"
              />
              <v-text-field
                v-model="form.password"
                :type="showPassword ? 'text' : 'password'"
                label="Password"
                prepend-icon="mdi-lock"
                :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
                @click:append="showPassword = !showPassword"
                autocomplete="on"
              />
              <v-card-actions>
                <v-btn type="submit" color="primary">Login</v-btn>
              </v-card-actions>
            </v-form>
          </v-card-text>
        </v-card>
      </v-row>
    </v-container>

    <v-snackbar v-model="snackbar.visible" :timeout="snackbar.timeout" :color="snackbar.color">
      {{ snackbar.text }}

      <template v-slot:action="{ attrs }">
        <v-btn color="white" text v-bind="attrs" @click="snackbar.visible = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>

  </div>
</template>

<script>

export default {
  auth: 'guest',
  name: "Login",
  layout: "LoggedOut",
  data() {
    return {
      snackbar: {
        visible: false,
        timeout: 2000,
        color: null,
        text: '',
      },
      form: {
        username: '',
        password: ''
      },
      showPassword: false,
    };
  },
  methods: {
    async userLogin() {
      try {
        await this.$auth.loginWith('local', {data: this.form})
          .then(async res => {

            // Decode JWT
            let jwt = res.data["JWT Token"]
            let jwtPayload = JSON.parse(atob(jwt.split('.')[1]))
            let userID = jwtPayload['id']

            // Get user data
            this.$axios.get(`/users/${userID}`)
              .then(user => {
                // this.$auth.$storage.setUniversal('user', user.data, true) // setting user in Vuex, cookies and localstorage
                this.$auth.setUser(user.data)

              })
              .catch(error => {
                console.log(error.message)
              })

          })
          .catch(error => {
            console.log(error.message)
          })
      } catch (error) {
        console.log(error.message)
      }
    }
  }
}
</script>

<style scoped>
/*.background {
  height: 100vh !important;
  background-image: linear-gradient(0deg, rgba(26, 26, 26, 0.5), rgba(26, 26, 26, 0.5)), url("../../assets/ljubljana.jpg");
  background-attachment: fixed;
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
}*/
</style>
