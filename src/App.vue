<template>
  <div id="app">
    <Navigation :user="user" @logout="logout" />
    <router-view
      class="container"
      :user="user"
      :meetings="meetings"
      :error="error"
      @logout="logout"
      @addMeeting="addMeeting"
      @deleteMeeting="deleteMeeting"
      @checkIn="checkIn"
    />
  </div>
</template>

<script>
import Navigation from "@/components/Navigation.vue";
import Firebase from "firebase";
/* eslint-disable */
import db from "./db.js";
/* eslint-enable */
export default {
  name: "app",
  data() {
    return {
      user: null,
      meetings: [],
      error: null

    };
  },
  components: {
    Navigation
  },
  methods: {
    logout: function() {
      Firebase.auth()
        .signOut()
        .then(() => {
          this.user = null;
          this.$router.push("login");
        });
    },
    addMeeting: function(payload) {
      db.collection("users")
        .doc(this.user.uid)
        .collection("meetings")
        .add({
          name: payload,
          createdAd: Firebase.firestore.FieldValue.serverTimestamp()
        });
    },
    deleteMeeting: function(payload) {
      db.collection("users")
        .doc(this.user.uid)
        .collection("meetings")
        .doc(payload)
        .delete();
    },
    checkIn: function(payload) {
      db.collection("users")
        .doc(payload.userID)
        .collection("meetings")
        .doc(payload.meetingID)
        .get()
        .then(doc => {
          if (doc.exists) {
            db.collection("users")
              .doc(payload.userID)
              .collection("meetings")
              .doc(payload.meetingID)
              .collection("attendees")
              .add({
                displayName: payload.displayName,
                email: payload.email,
                createdAt: Firebase.firestore.FieldValue.serverTimestamp()
              })
              .then(() => this.$router.push("/attendees/" + payload.userID+"/" + payload.meetingID));
          } else {
            this.error = "Sorry, no such meeting"
          }
        });
    }
  },
  mounted() {
    Firebase.auth().onAuthStateChanged(user => {
      if (user) {
        this.user = user;
        db.collection("users")
          .doc(this.user.uid)
          .collection("meetings")
          .onSnapshot(snapshot => {
            const snapData = [];
            snapshot.forEach(doc => {
              snapData.push({
                id: doc.id,
                name: doc.data().name
              });
            });
            this.meetings = snapData.sort((a, b) => {
              if (a.name.toLowerCase() < b.name.toLowerCase()) {
                return -1;
              } else {
                return 1;
              }
            });
          });
      }
    });
    // db.collection("users")
    //   .doc("1ZhT8gEjdgKcyXUBRErO")
    //   .get()
    //   .then(snapshot => {
    //     this.user = snapshot.data().name;
    //   });
  }
};
</script>

<style lang="scss">
$primary: #05b2dd;
@import "node_modules/bootstrap/scss/bootstrap";
</style>
