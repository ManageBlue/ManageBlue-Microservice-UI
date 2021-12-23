<template>
  <div>

    <v-container>

      <v-row align="center" justify="center">
        <v-col cols="10">
          <h1>Project: {{project.title}}</h1>
        </v-col>
      </v-row>

      <v-row align="center" justify="center">
        <v-col cols="10">
          <ProjectCard />
        </v-col>
      </v-row>

      <v-row align="center" justify="center">
        <v-col cols="10">
          <graph-ql-card/>
        </v-col>
      </v-row>

      <v-row align="center" justify="center">
        <v-col cols="10">
          <DeadlineCalendar/>
        </v-col>
      </v-row>


      <v-row align="center" justify="center">
        <v-col cols="10">
          <TaskTable/>
        </v-col>
      </v-row>

    </v-container>


  </div>
</template>

<script>
import ProjectCard from "~/components/ProjectCard";
import TaskTable from "~/components/TaskTable";
import DeadlineCalendar from "~/components/DeadlineCalendar";
import GraphQlCard from "~/components/graphQlCard";

export default {
  name: "Project",
  components: {GraphQlCard, DeadlineCalendar, TaskTable, ProjectCard},
  data: () => ({
    loading: true,

    project: {},
  }),
  methods:{
    getProjectById(){
      this.loading = true;

      this.$axios.get(`projects/api/v1/${this.$route.params.id}`)
        .then(response => {

          this.project = response.data;

        })
        .catch(error => {
          // logout if api call unAuthorized
          if(error.response && error.response.status === 401)
            this.globalUnAuthorization()
        })
        .finally(() => this.loading = false)

    }
  },
  mounted() {
    this.getProjectById()
  }
}
</script>

<style scoped>

</style>
