<template>
  <div id="app">
    <div id="nav">
      <a @click="logout" class="logout" v-if="authUser.displayName">Logout</a>
      <div class="user_profile" v-if="authUser.displayName">
        <div class="profile_img_cont">
          <img
            :src="authUser.photoURL"
            class="rounded-circle user_img"
          />
          <span class="online_icon"></span>
        </div>
        <div class="user_name">
          <span>{{ authUser.displayName }}</span>
        </div>
      </div>
    </div>
    <router-view />
  </div>
</template>

<script>
import firebase from "firebase";
export default {
  data() {
    return {
      authUser: {},
    };
  },
  created() {
    firebase.auth().onAuthStateChanged((user) => {
      if (user) {
        this.authUser = user;
      } else {
        this.authUser = {};
      }
    });
  },
  methods: {
    logout() {
      // Firestore
      // db.collection("users").doc(this.authUser.uid).update({
      //   uid: this.authUser.uid,
      //   name: this.authUser.displayName,
      //   loggedIn: false,
      //   last_changed: firebase.database.ServerValue.TIMESTAMP,
      // });
      var user = this.authUser;

      // Realtime Database
      firebase
        .auth()
        .signOut()
        .then(() => {
          // Sign-out successful.
          rtDb.ref("users/" + user.uid).update({loggedIn:false});
        })
        .catch((error) => {
          // An error happened.
        });
    },
  },
};
</script>
<style>
body,
html {
  height: 100%;
  margin: 0;
  background: #7f7fd5;
  background: -webkit-linear-gradient(to right, #91eae4, #86a8e7, #7f7fd5);
  background: linear-gradient(to right, #91eae4, #86a8e7, #7f7fd5);
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;
  margin: 0 0 30px;
  height: 80px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}

a.logout {
  float: right !important;
  cursor: pointer;
}

.user_profile {
  float: left;
}

.user_profile .user_name {
  float: right;
  margin-right: 10px;
  font-weight: 500;
}

.user_profile .profile_img_cont {
  float: left;
}

.profile_img_cont img {
  position: relative;
  height: 24px;
  width: 24px;
  margin: 0 6px 2px 0;
}
</style>
