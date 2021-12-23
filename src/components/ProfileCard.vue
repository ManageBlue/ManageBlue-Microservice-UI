<template>
  <v-card col="6" >

    <!-- card loader -->
    <div v-if="loading">
      <v-container>
        <v-row justify="center">
          <v-col cols="12" lg="6" xl="8">
            <v-skeleton-loader class="mx-auto" type="article, actions"/>
          </v-col>
        </v-row>
      </v-container>
    </div>

    <!-- card content -->
    <div v-else>
      <v-card-title>Profile</v-card-title>

      <v-container>
        <v-row justify="center">

          <v-col cols="12" >

            <v-form ref="form">

              <!-------------------------------------------------------------------->
              <h3>User data</h3>

              <v-text-field
                v-model="username"
                :counter="32"
                label="Username"
                required
                :rules="shortLengthRequiredRules"
              ></v-text-field>

              <v-text-field
                v-model="firstName"
                :counter="32"
                label="First name"
                required
                :rules="shortLengthRequiredRules"
              ></v-text-field>

              <v-text-field
                v-model="lastName"
                :counter="32"
                label="Last name"
                required
                :rules="shortLengthRequiredRules"
              ></v-text-field>

              <v-text-field
                v-model="email"
                :counter="128"
                label="E-mail"
                required
                :rules="emailRules"
              ></v-text-field>

              <v-btn color="primary" text class="mr-4" @click="submit">
                Update
              </v-btn>

            </v-form>
          </v-col>
        </v-row>
      </v-container>

    </div>

    <v-snackbar v-model="snackbar.visible" :timeout="snackbar.timeout" :color="snackbar.color">
      {{ snackbar.text }}

      <template v-slot:action="{ attrs }">
        <v-btn color="white" text v-bind="attrs" @click="snackbar.visible = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>

  </v-card>
</template>

<script>

export default {
  name: 'ProfileCard',
  props: {
    id: String,
  },
  data: () => ({
    tab: null,
    loading: true,
    snackbar: {
      visible: false,
      timeout: 2000,
      text: '',
    },
    username: '',
    firstName: '',
    lastName: '',
    email: '',

    // validation rules
    shortLengthRequiredRules: [
      v => !!v || 'Field required',
      v => v.length <= 32 || 'Field can not be longer than 32 characters',
    ],
    emailRules: [
      v => !!v || 'E-mail address is required',
      v => v.length <= 128 || 'Field can not be longer than 128 characters',
      v => /^(([^<>()[\]\\.,;:\s@']+(\.[^<>()\\[\]\\.,;:\s@']+)*)|('.+'))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/.test(v) || 'E-mail address must be valid',
    ],
  }),
  methods: {
    submit() {
      if (this.$refs.form.validate()) {

        // create an object with updated user data
        let updatedUser = {
          username: this.username,
          firstName: this.firstName,
          lastName: this.lastName,
          email: this.email,
        }

        this.$axios.put('users/api/v1/' + this.id, updatedUser)
          .then(() => {

            this.snackbar.text = "User updated"
            this.snackbar.color = null;
            this.snackbar.visible = true;
          })
          .catch(error => {
            this.snackbar.text = "Error while updating user"
            this.snackbar.color = "error";
            this.snackbar.visible = true;


          })
      }
    }

  }
  ,
  mounted() {
    this.$axios.get('users/api/v1/' + this.id)
      .then(response => {

        let user = response.data

        // fill fields with fetched data
        this.username = user.username;
        this.firstName = user.firstName;
        this.lastName = user.lastName;
        this.email = user.email;

      })
      .catch(error => {


      })
      .finally(() => this.loading = false)
  }
}

</script>

<style scoped>

</style>
