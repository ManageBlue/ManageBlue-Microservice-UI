<template>
  <div>

    <v-container>

      <v-row align="center" justify="center">

        <!-- title -->
        <v-col cols="9">
          <h1>Dashboard</h1>
        </v-col>

        <!-- new project button -->
        <v-col cols="1">
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn icon v-bind="attrs" v-on="on" @click="openCreateDialog">
                <v-icon x-large color="primary">mdi-database-plus</v-icon>
              </v-btn>
            </template>
            <span>New project</span>
          </v-tooltip>
        </v-col>

      </v-row>

      <v-row align="center" justify="center" v-for="(project, i) in projects" :key="i">
        <v-col cols="10">


          <!-- project card -->
          <v-card class="mx-auto" hover nuxt :to="{ name: 'project-id', params: { id: project._id }}">

            <v-card-title> {{ project.title }}</v-card-title>
            <v-card-subtitle>{{ project.description }}</v-card-subtitle>
            <v-divider/>

            <v-list dense>
              <v-list-item v-for="(member, i) in project.members" :key="i">

                <v-list-item-icon>
                  <v-icon>mdi-account</v-icon>
                </v-list-item-icon>

                <v-list-item-content>
                  <v-list-item-subtitle>{{ member.firstName }} {{ member.lastName }}</v-list-item-subtitle>
                </v-list-item-content>

                <!--
                <v-list-item-avatar>
                  <v-img src="https://images-wixmp-ed30a86b8c4ca887773594c2.wixmp.com/f/2aba9247-2ca7-423b-832b-663ce34db900/dcwmqtq-084c7a12-e3bd-409f-9b49-a0bca9a29514.png/v1/fill/w_850,h_940,strp/big_chungus_png_hq_by_yellogre_dcwmqtq-pre.png?token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1cm46YXBwOjdlMGQxODg5ODIyNjQzNzNhNWYwZDQxNWVhMGQyNmUwIiwiaXNzIjoidXJuOmFwcDo3ZTBkMTg4OTgyMjY0MzczYTVmMGQ0MTVlYTBkMjZlMCIsIm9iaiI6W1t7ImhlaWdodCI6Ijw9MTQxNiIsInBhdGgiOiJcL2ZcLzJhYmE5MjQ3LTJjYTctNDIzYi04MzJiLTY2M2NlMzRkYjkwMFwvZGN3bXF0cS0wODRjN2ExMi1lM2JkLTQwOWYtOWI0OS1hMGJjYTlhMjk1MTQucG5nIiwid2lkdGgiOiI8PTEyODAifV1dLCJhdWQiOlsidXJuOnNlcnZpY2U6aW1hZ2Uub3BlcmF0aW9ucyJdfQ.e5EQVLJr0zgYh6kgkt1lzQqKqCBFI6kMyku7cJbvRrA"></v-img>
                </v-list-item-avatar>
                -->
              </v-list-item>
            </v-list>
          </v-card>


        </v-col>
      </v-row>

    </v-container>

    <!-- new project dialog -->
    <v-dialog transition="dialog-bottom-transition" persistent max-width="600" v-model="dialog">
      <v-card>

        <v-card-title class="headline">
          New Project
        </v-card-title>

        <v-card-text>
          <p>Fill in required field and click "Create" to create a new project.</p>

          <!-- input form -->
          <v-form ref="form">

            <v-container>

              <!-- input field for title -->
              <v-row align="center" justify="center">
                <v-col cols="12">
                  <v-text-field
                    v-model="projectModel.title"
                    :rules="shortLengthRules"
                    :counter="64"
                    label="Project title"
                  ></v-text-field>
                </v-col>
              </v-row>

              <!-- input field for note -->
              <v-row align="center" justify="center">
                <v-col cols="12">
                  <v-textarea
                    v-model="projectModel.description"
                    :counter="1024"
                    label="Project description"
                    :rules="longLengthRules"
                    auto-grow
                  ></v-textarea>
                </v-col>
              </v-row>

              <!-- input selector for contributor -->
              <v-row>
                <v-col>
                  <v-select
                    v-model="projectModel.members"
                    :items="allUsers"
                    item-text="display"
                    item-value="_id"
                    label="Members"
                    :rules="selectRules"
                    multiple
                  ></v-select>
                </v-col>
              </v-row>

            </v-container>
          </v-form>
        </v-card-text>

        <!-- card actions for creating project -->
        <v-card-actions>

          <v-spacer/>

          <v-btn color="primary" text plain @click="closeCreateDialog">
            Cancel
          </v-btn>

          <v-btn color="primary" text plain @click="createProject">
            Create
          </v-btn>

        </v-card-actions>
      </v-card>
    </v-dialog>

  </div>
</template>

<script>
import ProjectCard from "~/components/ProjectCard";

export default {
  components: {ProjectCard},
  data: () => ({
    loading: false,
    dialog: false,
    projects: [],
    allUsers: [],

    projectModel: {
      "title": null,
      "description": null,
      "members": [],
      "active": true,
    },

    // rules for input fields
    shortLengthRules: [
      v => !!v || 'Field required',
      v => !v || v.length <= 64 || 'Field can not exceed 64 characters',
    ],
    longLengthRules: [
      v => !!v || 'Field required',
      v => !v || v.length <= 1024 || 'Field can not exceed 1024 characters',
    ],
    numberRules: [
      v => (v === 0 || !!v) || 'Field required',
      v => v > 0 || "Field value can not be lower than 1",
      v => v <= 10000 || "Vrednost polja ne sme presegati 10.000",
    ],
    selectRules: [(v) => !!v || "Field required"],
    userDict: {
      "615d6e3f148db6592a35fe4d":{
        firstName: "Matej",
        lastName: "Horvat"
      },
      "615dbe8354c1336bde16918e":{
        firstName: "Lan",
        lastName: "Sovinc"
      }
    }
  }),
  methods: {

    // open dialog after fetching all users
    async openCreateDialog() {
      this.getUsers().finally(() => {
        this.dialog = true
      })
    },
    // reset form before closing dialog
    closeCreateDialog() {
      this.$refs.form.reset()
      this.dialog = false;
    },

    createProject() {

      // validate the form's correctness
      if (this.$refs.form.validate()) {

        this.$axios.post('projects/api/v1', this.projectModel)
          .then(() => {

            /*this.snackbar.text = "Nepremičnina ustvarjena"
                this.snackbar.color = null;
                this.snackbar.visible = true;*/

            //fetch again after updating
            this.getProjects()

          })
          .catch(error => {
            /*this.snackbar.text = "Napaka pri ustvarjanju nepremičnine"
            this.snackbar.color = "error";
            this.snackbar.visible = true;*/

            // logout if api call unAuthorized
            if (error.response && error.response.status === 401)
              this.globalUnAuthorization()
          })
          .finally(() => {
            this.dialog = false
          })
      }
    },
    getUsers() {
      return this.$axios.get('users/api/v1')
        .then(response => {

          this.allUsers = response.data.map(user => {
            return {
              display: user.firstName + ' ' + user.lastName,
              _id: user._id,
            }
          })

        })
        .catch(error => {
          // logout if api call unAuthorized
          if (error.response && error.response.status === 401)
            this.globalUnAuthorization()
        })
    },

    getProjects() {
      this.loading = true;

      this.$axios.get('projects/api/v1')
        .then(response => {

          this.projects = response.data;

          for (let i in this.projects){

            this.projects[i].members = this.projects[i].members.map(member => {

              return {
                id: member,
                firstName: this.userDict[member].firstName,
                lastName: this.userDict[member].lastName
              }
            })
          }

        })
        .catch(error => {
          // logout if api call unAuthorized
          if (error.response && error.response.status === 401)
            this.globalUnAuthorization()
        })
        .finally(() => this.loading = false)

    }
  },
  mounted() {
    this.getProjects()
  }
}
</script>
