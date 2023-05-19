<template>
  <div>
    <NewMeetingForm @added="addNewMeeting($event)"></NewMeetingForm>
    <span v-if="meetings.length == 0">Brak zaplanowanych spotkań.</span>

    <h3 v-else>Zaplanowane zajęcia ({{ meetings.length }})</h3>
    <MeetingsList :meetings="meetings"
                  :username="username"
                  @attend="addMeetingParticipant($event)"
                  @unattend="removeMeetingParticipant($event)"
                  @delete="deleteMeeting($event)"></MeetingsList>
  </div>
</template>

<script>
import NewMeetingForm from "./NewMeetingForm";
import MeetingsList from "./MeetingsList";
import axios from "axios";

export default {
  components: {NewMeetingForm, MeetingsList},
  props: {username: String},
  data() {
    return {
      meetings: [],
      message: '',
      meetingCreated: false,
    };
  },
  created() {
      this.loadMeetings();
  },
  methods: {
      loadMeetings() {
          axios
              .get("/api/meetings", { params: { username: this.username } })
              .then(response => {
                  this.meetings = response.data;
              })
              .catch(error => {
                  console.log("Nie udało się załadować spotkań.");
              });
      },
      addNewMeeting(meeting) {
          axios.post('/api/meetings', meeting)
              .then(response => {
                  console.log('Dodano spotaknie.');
                  this.meetings.push(meeting);
              })
              .catch(response => console.log('Nie udało się dodać spotkania'))
      },
      addMeetingParticipant(meeting) {
          this.meetings.at(this.meetings.indexOf(meeting)).participants.push(this.username);
          axios.post('/api/meetings/' + meeting.id + '/participants', {login:this.username} )
              .then(response => {
                  console.log('Dodano uczestnika.');
              })
              .catch(response => console.log('Nie udało się dodać uczestnika'))
      },
    removeMeetingParticipant(meeting) {
        this.meetings.at(this.meetings.indexOf(meeting)).participants.push(this.username);
        axios.post('/api/meetings/' + meeting.id + '/participants', {login:this.username} )
            .then(response => {
                console.log('Dodano uczestnika do spotkania.');
            })
            .catch(response => console.log('Nie udało się dodać uczestnika do spoktania'))
    },
    deleteMeeting(meeting) {
        axios.delete('/api/meetings/' + meeting.id)
            .then(response => {
                console.log('Usunięto spotkanie.');
            })
            .catch(response => console.log('Nie udało się usunąć spotkania'))
        this.meetings.splice(this.meetings.indexOf(meeting), 1);
    },
  }
}
</script>

<style>
.alert {
    padding: 5px;
    border: 2px solid green;
    background: lightgreen;
    font-size: 2em;
    color: darkgreen;
    text-align: center;
}

.alert-red {
    padding: 5px;
    font-size: 2em;
    color: darkred;
    border: 2px solid red;
    background: lightcoral;
    text-align: center;
}
</style>
