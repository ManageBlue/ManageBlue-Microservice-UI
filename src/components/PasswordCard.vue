<template>
  <v-card>

    <v-card-title>New Password</v-card-title>

    <v-container>
      <v-row justify="center">
        <v-col>

          <v-form ref="form">
            <v-text-field
              v-model="password1"
              :counter="128"
              label="New Password"
              :rules="passwordRule"
              :type="showPassword1 ? 'text' : 'password'"
              :append-icon="showPassword1 ? 'mdi-eye' : 'mdi-eye-off'"
              @click:append="showPassword1 = !showPassword1"
              autocomplete="on"
            ></v-text-field>
            <v-text-field
              v-model="password2"
              :counter="128"
              label="Repeat new password"
              required
              :rules=[matchRule]
              :type="showPassword2 ? 'text' : 'password'"
              :append-icon="showPassword2 ? 'mdi-eye' : 'mdi-eye-off'"
              @click:append="showPassword2 = !showPassword2"
              autocomplete="on"
            ></v-text-field>
          </v-form>

          <v-btn color="primary" text class="mr-4" @click="submit">
            Change password
          </v-btn>

        </v-col>
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

  </v-card>
</template>

<script>
export default {
  name: "PasswordCard",
  props: {
    id: String,
  },
  data: () => ({
    showPassword1: false,
    showPassword2: false,
    snackbar: {
      visible: false,
      timeout: 2000,
      text: '',
      color: null
    },
    password1: '',
    password2: '',
    // validation rules
    passwordRule: [
      v => !!v || 'Field Required',
      v => v.length <= 128 || 'Password can not be longer than 128 characters',
      v => /(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,}/.test(v) || 'Password must contain at leas one lowercase letter, one uppercase letter, one number, one special character and is at least 8 characters long',
    ],
  }),
  computed: {
    matchRule() {
      return () => (this.password1 === this.password2) || 'Passwords must match'
    }
  },
  methods: {
    submit() {
      if (this.$refs.form.validate()) {
        this.$axios.put('users/pass/api/v1/' + this.id, {password: this.password1})
          .then(() => {
            this.snackbar.text = "Password updated"
            this.snackbar.visible = true;
            this.snackbar.color = null;

          })
          .catch(error => {
            this.snackbar.text = "Error while changing password"
            this.snackbar.visible = true;
            this.snackbar.color = "error";

          })
      }
    }
  }
}
</script>

<style scoped>

</style>
