<template>

  <v-container>

    <!-- user data card -->
    <v-row align="center" justify="center">
      <v-col cols="10">
        <profile-card :id="userId"/>
      </v-col>
    </v-row>


  </v-container>

</template>

<script>
import ProfileCard from "~/components/ProfileCard";
import PasswordCard from "~/components/PasswordCard";

export default {
  name: "profile",
  components: {PasswordCard, ProfileCard},
  computed: {
    // get user id from nuxt auth
    userId() {
      return "615dbe8354c1336bde16918e"
    }
  },
}
</script>

<style scoped>

</style>
