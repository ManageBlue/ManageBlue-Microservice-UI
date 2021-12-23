<template>
  <div>

    <!-- tasks table card-->
    <v-card>

      <v-card-title>
        Tasks
        <v-spacer/>

        <!-- button for creating tasks-->
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-btn icon v-bind="attrs" v-on="on" @click="openDialogCreate">
              <v-icon color="primary">mdi-clipboard-plus</v-icon>
            </v-btn>
          </template>
          <span>New task</span>
        </v-tooltip>

      </v-card-title>

      <!-- task data table-->
      <v-data-table
        :headers="headers"
        :items="tasks"
        class="elevation-1"
        :loading="loading"
        loading-text="Loading... please wait a moment"
        :page="page"
        :server-items-length="totalTasks"
        :options.sync="options"
        :footer-props="{
             'items-per-page-options': [16],
        }"
        disable-sort
        :expanded.sync="expanded"
        item-key="_id"
        show-expand
      >

        <!-- expander activator -->
        <template v-slot:expanded-item="{ headers, item }">
          <td :colspan="headers.length">
            {{ item.note }}
          </td>
        </template>

        <!-- completed -->
        <template v-slot:[`item.completed`]="{ item }">
          <div class="p-2">
            <v-icon v-if="item.completed" color="primary">mdi-check</v-icon>
            <v-icon v-else color="error">mdi-close</v-icon>
          </div>
        </template>

        <!-- paid -->
        <template v-slot:[`item.paid`]="{ item }">
          <div class="p-2">
            <v-icon v-if="item.paid" color="primary">mdi-check</v-icon>
            <v-icon v-else color="error">mdi-close</v-icon>
          </div>
        </template>

        <!-- row actions -->
        <template v-slot:[`item.actions`]="{ item }">

          <!-- action EDIT -->
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn icon v-bind="attrs" v-on="on" @click="openDialogEdit(item)">
                <v-icon color="primary">mdi-clipboard-edit</v-icon>
              </v-btn>
            </template>
            <span>Edit task</span>
          </v-tooltip>

        </template>

      </v-data-table>
    </v-card>

    <!-- delete task dialog -->
    <v-row justify="center">
      <v-col cols="10">
        <v-dialog transition="dialog-bottom-transition" max-width="600" v-model="dialogDelete">
          <v-card>
            <v-card-title class="headline">
              Delete the task?
            </v-card-title>

            <v-card-text>
              Are you sure you want to delete the task?
            </v-card-text>

            <!-- dialog buttons -->
            <v-card-actions>
              <v-spacer></v-spacer>

              <v-btn color="primary" text plain @click="dialogDelete = false">
                Cancel
              </v-btn>

              <v-btn color="error" text plain @click="deleteTask">
                Delete Task
              </v-btn>

            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-col>
    </v-row>

    <!-- creat edit task dialog -->
    <v-row justify="center">
      <v-col cols="10">
        <v-dialog transition="dialog-bottom-transition" persistent max-width="600" v-model="dialogCE">
          <v-card>

            <!-- different titles depending on creating or editing -->
            <v-card-title class="headline" v-if="editingTask">
              Edit Task
            </v-card-title>

            <v-card-title class="headline" v-else>
              New Task
            </v-card-title>

            <!-- different card text depending on creating or editing -->
            <v-card-text>
              <p v-if="editingTask">Fill in required field and click "Confirm edit" to edit an existing task.</p>
              <p v-else>Fill in required field and click "Create" to create a new task.</p>

              <!-- input form -->
              <v-form ref="form">

                <v-container>

                  <!-- input field for title -->
                  <v-row align="center" justify="center">
                    <v-col cols="12">
                      <v-text-field
                        v-model="taskModel.title"
                        :rules="shortLengthRules"
                        :counter="64"
                        label="Task title"
                      ></v-text-field>
                    </v-col>
                  </v-row>

                  <!-- input field for note -->
                  <v-row align="center" justify="center">
                    <v-col cols="12">
                      <v-textarea
                        v-model="taskModel.note"
                        :counter="1024"
                        label="Task description"
                        :rules="longLengthRules"
                        auto-grow
                      ></v-textarea>
                    </v-col>
                  </v-row>

                  <!-- input fields for hours & hourly rate -->
                  <v-row align="center" justify="center">
                    <v-col cols="6">
                      <v-text-field
                        v-model.number="taskModel.hours"
                        type="number"
                        label="Hours"
                        :rules="numberRules"
                        prepend-inner-icon="mdi-clock-outline"
                      ></v-text-field>
                    </v-col>

                    <v-col cols="6">
                      <v-text-field
                        v-model.number="taskModel.hourlyRate"
                        type="number"
                        label="Hourly rate"
                        :rules="numberRules"
                        prepend-inner-icon="mdi-currency-eur"
                      ></v-text-field>
                    </v-col>
                  </v-row>

                  <!-- input selector for contributor -->
                  <v-row>
                    <v-col>
                      <v-select
                        v-model="taskModel.contributor"
                        :items="possibleContributors"
                        item-text="display"
                        item-value="_id"
                        label="Contributor"
                        :rules="selectRules"
                      ></v-select>
                    </v-col>
                  </v-row>

                  <!-- checkboxes for completion & payment -->
                  <v-row>
                    <v-col>
                      <p>Completed</p>
                      <v-checkbox color="primary" v-model="taskModel.completed" label="Completed"/>
                    </v-col>
                    <v-col>
                      <p>Paid</p>
                      <v-checkbox color="primary" v-model="taskModel.paid" label="Paid"/>
                    </v-col>
                  </v-row>

                  <!-- input field for date selector -->
                  <v-row>
                    <v-col>
                      <v-menu
                        v-model="menu"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        transition="scale-transition"
                        offset-y
                        min-width="auto"
                      >
                        <template v-slot:activator="{ on, attrs }">
                          <v-text-field
                            v-model="taskModel.date"
                            label="Date of task"
                            prepend-icon="mdi-calendar"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-date-picker
                          v-model="taskModel.date"
                          @input="menu = false"
                        ></v-date-picker>
                      </v-menu>
                    </v-col>
                  </v-row>

                </v-container>
              </v-form>
            </v-card-text>

            <!-- card actions for editing task -->
            <v-card-actions v-if="editingTask">

              <v-btn color="error" text plain @click="dialogDelete = true">
                Delete
              </v-btn>

              <v-spacer/>

              <v-btn color="primary" text plain @click="closeDialogCE">
                Cancel
              </v-btn>

              <v-btn color="primary" text plain @click="updateTask">
                Update
              </v-btn>

            </v-card-actions>

            <!-- card actions for creating task -->
            <v-card-actions v-else>

              <v-spacer/>

              <v-btn color="primary" text plain @click="closeDialogCE">
                Cancel
              </v-btn>

              <v-btn color="primary" text plain @click="createTask">
                Create
              </v-btn>

            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-col>
    </v-row>

  </div>
</template>

<script>
export default {
  name: "TaskTable",
  data: () => ({
    loading: true,
    dialogDelete: null,
    dialogCE: null,
    editingTask: false,
    selectedTaskId: null,
    menu: false,

    // headers for data table
    headers: [
      {text: 'Title', value: 'title'},
      {text: 'Hours', value: 'hours'},
      {text: 'Rate', value: 'hourlyRate'},
      {text: 'Contributor', value: 'contributorDisplay'},
      {text: 'Date', value: 'date'},
      {text: 'Completed', value: 'completed'},
      {text: 'Paid', value: 'paid'},
      {text: 'Actions', value: 'actions'},
      {text: '', value: 'data-table-expand'},
    ],
    // array of expanded fields
    expanded: [],

    // pagination
    page: 1,
    totalTasks: 0,
    options: {},

    // model for input fields
    taskModel: {
      "title": null,
      "note": "",
      "hours": null,
      "hourlyRate": null,
      "date": "",
      "contributor": "",
      "completed": false,
      "paid": false,
      "project": ""
    },
    // array of retrieved tasks
    tasks: [],

    // array of retrieved project members
    possibleContributors: [],

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

    // opens dialog for creating task
    // sets the task's date and project
    // gets all members of project for selecting the contributor before opening dialog
    openDialogCreate() {
      this.editingTask = false
      this.taskModel.date = (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10)
      this.taskModel.project = this.$route.params.id

      this.getMembersOfProject().finally(() => {
        this.dialogCE = true
      })
    },

    // opens dialog for editing task
    // fills in the task's details & sets the variable for api access by id
    // gets all members of project for selecting the contributor before opening dialog
    async openDialogEdit(selectedRow) {
      this.editingTask = true

      this.selectedTaskId = selectedRow._id

      this.taskModel = {...selectedRow}
      delete this.taskModel._id;
      delete this.taskModel.createdAt;
      delete this.taskModel.updatedAt;

      this.taskModel.project = this.$route.params.id

      this.getMembersOfProject().finally(() => {
        this.dialogCE = true
      })
    },

    // closes dialog for creating and editig tasks
    // clears the dialog's form
    closeDialogCE() {
      this.$refs.form.reset()
      this.taskModel.completed = false
      this.taskModel.paid = false
      this.dialogCE = false;
    },

    // create the task by api call
    createTask() {

      // validate the form's correctness
      if (this.$refs.form.validate()) {

        this.$axios.post('tasks/api/v1/', this.taskModel)
          .then(() => {

            /*this.snackbar.text = "Nepremičnina ustvarjena"
                this.snackbar.color = null;
                this.snackbar.visible = true;*/

            // fetch again after updating
            this.getTasks();

          })
          .catch(error => {
            /*this.snackbar.text = "Napaka pri ustvarjanju nepremičnine"
            this.snackbar.color = "error";
            this.snackbar.visible = true;*/

            // logout if api call unAuthorized
            if(error.response && error.response.status === 401)
              this.globalUnAuthorization()
          })
          .finally(() => {
            this.closeDialogCE()
          })
      }
    },

    // update the task by it's selected id
    updateTask() {

      // validate the form's correctness
      if (this.$refs.form.validate()) {

        this.$axios.put('tasks/api/v1/' + this.selectedTaskId, this.taskModel)
          .then(() => {

            this.closeDialogCE()

            /*this.snackbar.text = "Nepremičnina ustvarjena"
                this.snackbar.color = null;
                this.snackbar.visible = true;*/

            // fetch again after updating
            this.getTasks();
          })
          .catch(error => {
            /*this.snackbar.text = "Napaka pri ustvarjanju nepremičnine"
            this.snackbar.color = "error";
            this.snackbar.visible = true;*/

            // logout if api call unAuthorized
            if(error.response && error.response.status === 401)
              this.globalUnAuthorization()
          })
          .finally(() => {
            this.selectedTaskId = null
            this.closeDialogCE()
          })
      }

    },

    // delete the task by it's selected id
    deleteTask() {

      this.$axios.delete('tasks/api/v1/' + this.selectedTaskId)
        .then(() => {

          /*this.snackbar.text = "Nepremičnina ustvarjena"
              this.snackbar.color = null;
              this.snackbar.visible = true;*/

          // fetch again after updating
          this.getTasks();

        })
        .catch(error => {
          /*this.snackbar.text = "Napaka pri ustvarjanju nepremičnine"
          this.snackbar.color = "error";
          this.snackbar.visible = true;*/
          // logout if api call unAuthorized
          if(error.response && error.response.status === 401)
            this.globalUnAuthorization()
        })
        .finally(() => {
          this.selectedTaskId = null
          this.dialogDelete = false
          this.closeDialogCE()
        })

    },

    // return all tasks
    getTasks() {
      this.loading = true;

      // pagination
      const {page} = this.options;
      let pageNumber = page - 1;

      this.$axios.get(`tasks/api/v1/?page=${pageNumber}&project=${this.$route.params.id}`)
        .then(response => {

          this.tasks = response.data.tasks.map(task => {
            console.log(task)
            return {
              ...task,
              contributorDisplay: this.userDict[task.contributor].firstName + ' ' + this.userDict[task.contributor].lastName,
              date: task.date.substring(0, 10),
            }
          })

          // set total task count
          this.totalTasks = response.data.count;

        })
        .catch(error => {

          // logout if api call unAuthorized
          if(error.response && error.response.status === 401)
            this.globalUnAuthorization()
        })
        .finally(() => this.loading = false)
    },

    // return only members of project & build display name for selector
    getMembersOfProject() {
      return this.$axios.get('projects/api/v1/' + this.$route.params.id)
        .then(response => {

          this.possibleContributors = response.data.members.map(member => {
            return {
              display: this.userDict[member].firstName + ' ' + this.userDict[member].lastName,
              _id: member,
            }
          })

        })
        .catch(error => {
          // logout if api call unAuthorized
          if(error.response && error.response.status === 401)
            this.globalUnAuthorization()
        })
    },
  },
  // required for pagination
  // kako dela? nimam pojma
  watch: {
    options: {
      handler() {
        this.getTasks();
      },
    },
    deep: true,
  },

}
</script>

<style scoped>

</style>
