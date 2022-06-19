<template>
  <div>
    <new-meeting-form @added="addNewMeeting($event)"></new-meeting-form>

    <span v-if="meetings.length == 0">
               Brak zaplanowanych spotkań.
           </span>
    <h3 v-else>
      Zaplanowane zajęcia ({{ meetings.length }})
    </h3>

    <meetings-list :meetings="meetings"
                   :username="username"
                   @attend="addMeetingParticipant($event)"
                   @unattend="removeMeetingParticipant($event)"
                   @delete="deleteMeeting($event)"></meetings-list>
  </div>
</template>

<script>
    import NewMeetingForm from "./NewMeetingForm";
    import MeetingsList from "./MeetingsList";

    export default {
      mounted() {
        this.$http.get('meetings')
            .then((response) => {
              console.log('Zaimportowano listę spotkań.');
              this.meetings = response.data;
            })
            .catch(response => console.log('Nie udało się pobrać spotkań: ' + response.status));
      },
        components: {NewMeetingForm, MeetingsList},
        props: ['username'],
        data() {
            return {
                meetings_list: []
            };
        },
        computed: {
          meetings: {
            get() {
            return this.meetings_list;
            },
            set(meetings) {
              this.meetings_list = meetings;
            }
          }
        },
        methods: {
          fetchMeetings(){
            this.$http.get('meetings')
                .then(response => {
                  this.meetings = response.body;
                })
                .catch(response => {
                  console.log("Nie udało się pobrać spotkań.");
                });

          },
            addNewMeeting(meeting) {
                this.meetings.push(meeting);
                this.$http.post('meetings', meeting)
                  .then(() => {
                    console.log('Spotkanie utworzone.');
                    this.fetchMeetings();
                  })
                  .catch(response => console.log('Nie udało się utworzyć spotkania: ' + response.status));
            },
            addMeetingParticipant(meeting) {
                meeting.participants.push(this.username);
                this.$http.post('meetings/' + meeting.id + '/participants', {login:this.username})
              .then(response => {
                console.log('Dodano użytkownika do spotkania.');
                this.fetchMeetings();
              })
              .catch(response => {
                console.log('Nie udało się dodać użytkownika do spotkania: ' + response.status);
              });
            },
            removeMeetingParticipant(meeting) {
                meeting.participants.splice(meeting.participants.indexOf(this.username), 1);
                this.$http.delete('meetings/' + meeting.id + '/participants/' + this.username)
                    .then(response => {
                      console.log('Usunięto użytkownika ze spotkania.');
                      this.fetchMeetings();
                    })
                    .catch(response => {
                      console.log('Nie udało się usunąć użytkownika ze spotkania: ' + response.status);
                    });
            },
            deleteMeeting(meeting) {
                this.meetings.splice(this.meetings.indexOf(meeting), 1);
                this.$http.delete('meetings/' + meeting.id)
                    .then(response => {
                      console.log('Usunięto spotkanie.');
                    })
                    .catch(response => {
                        console.log('Nie udało usunąć spotkania: ' + response.status);
            });
          }
        }
    }
</script>
