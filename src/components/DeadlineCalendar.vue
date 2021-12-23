<template>
  <v-card>
    <v-card-title>Deadlines</v-card-title>
    <v-card-subtitle>
      <v-toolbar
        flat
      >
        <v-btn

          class="mr-4"
          color="grey darken-3"
          @click="setToday"
        >
          Today
        </v-btn>
        <v-btn
          icon
          text
          small
          color="grey darken-2"
          @click="prev"
        >
          <v-icon small>
            mdi-chevron-left
          </v-icon>
        </v-btn>
        <v-btn
          icon
          text
          small
          color="grey darken-2"
          @click="next"
        >
          <v-icon small>
            mdi-chevron-right
          </v-icon>
        </v-btn>
        <v-toolbar-title v-if="$refs.calendar">
          {{ $refs.calendar.title }}
        </v-toolbar-title>

      </v-toolbar>
    </v-card-subtitle>

    <v-card-text>
      <v-calendar
        ref="calendar"
        v-model="focus"

        :events="deadlines"
        color="primary"
        type="month"

      ></v-calendar>
    </v-card-text>

  </v-card>
</template>

<script>
export default {
  name: "DeadlineCalendar",
  data: () => ({
    focus: '',
    today: '2019-01-08',
    deadlines: [
      {id: "1", name: "RSO zagovor 1", start: "2022-01-04 14:15", projectId: "61c4580b87283c4e5b3a01cb"},
      {id: "2", name: "RSO zagovor 2", start: "2022-01-11 14:45", projectId: "61c4580b87283c4e5b3a01cb"},
      {id: "3", name: "RSO kolokvij", start: "2022-01-11 12:00", projectId: "61c4580b87283c4e5b3a01cb"},
      {id: "4", name: "MAT kolokvij", start: "2022-01-13 19:00", projectId: "61c4617a87283c4e5b3a01f3"},
    ],
  }),
  methods: {
    setToday() {
      this.focus = ''
    },
    prev() {
      this.$refs.calendar.prev()
    },
    next() {
      this.$refs.calendar.next()
    },
  },
  mounted() {
    this.today=Date.now()
    this.$refs.calendar.prev()
    this.setToday()
  }
}
</script>

<style scoped>

</style>
