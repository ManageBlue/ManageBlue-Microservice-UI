<template>
  <v-card>
    <v-card-title>
      Financing
    </v-card-title>

    <v-card-subtitle>Finances of each member in project</v-card-subtitle>

    <v-divider/>


    <v-list dense v-if="this.$route.params.id === '61c4580b87283c4e5b3a01cb'">
      <v-list-item v-for="(member, i) in financeData.members" :key="i">

        <v-list-item-icon>
          <v-icon>mdi-account</v-icon>
        </v-list-item-icon>

        <v-list-item-content>
          <v-list-item-title>{{ member.firstName }} {{ member.lastName }}</v-list-item-title>
          <v-list-item-subtitle>total hours: {{ member.totalHours }}</v-list-item-subtitle>
          <v-list-item-subtitle>total paid: {{ member.totalPaid }}€</v-list-item-subtitle>
          <v-list-item-subtitle>total owed: {{ member.totalOwed }}€</v-list-item-subtitle>
        </v-list-item-content>

      </v-list-item>

    </v-list>

    <v-card-text v-else>Data unavailable</v-card-text>


  </v-card>
</template>

<script>
import error from "~/layouts/error";

export default {
  name: "graphQlCard",
  data: () => ({
    financeData: {}
  }),
  mounted() {

    let graphqlQuery = {
      "query": `{
          projects{
              id
              members{
                  firstName
                  firstName
                  lastName
                  totalHours
                  totalPaid
                  totalOwed
              }
              totalHours
              totalPaid
              totalOwed
          }
      }`,
    }


    this.$axios.post('graphql/api/v1', JSON.stringify(graphqlQuery), {
      headers: {
        "content-type": "application/json",
      }
    })
      .then((res) => {
        this.financeData = res.data.data.projects[0]
        //console.log(this.financeData)

      }).catch(error => {
      console.log(error.response.data)
    })
  }
}
</script>

<style scoped>

</style>
