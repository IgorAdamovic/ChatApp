<template>
  <div class="container-fluid h-100">
    <div class="row justify-content-center h-100">
      <div class="col-md-4 col-xl-3 chat">
        <div class="card mb-sm-3 mb-md-0 contacts_card">
          <div class="card-header">
            <div class="input-group">
              <input
                type="text"
                placeholder="Search..."
                name=""
                class="form-control search"
              />
              <div class="input-group-prepend">
                <span class="input-group-text search_btn"
                  ><i class="fas fa-search"></i
                ></span>
              </div>
            </div>
          </div>
          <div class="card-body contacts_body">
            <div class="contacts">
              <li
                v-for="(user, index) in users"
                :key="index"
                @click="readMessages(user.uid)"
                :class="user.name === friend.name ? 'active' : ''"
              >
                <div class="d-flex bd-highlight">
                  <div class="img_cont">
                    <img
                      :src="user.profile_picture"
                      class="rounded-circle user_img"
                    />
                    <span class="online_icon" v-if="user.loggedIn"></span>
                    <span class="online_icon offline" v-else></span>
                  </div>
                  <div class="user_info">
                    <span>{{ user.name }}</span>
                    <p v-if="user.loggedIn">{{ user.name }} is online</p>
                    <p v-else>{{ user.name }} is offline</p>
                  </div>
                </div>
              </li>
            </div>
          </div>
          <div class="card-footer"></div>
        </div>
      </div>
      <div
        class="col-md-8 col-xl-6 chat"
        :class="[isSelected ? 'visible' : 'hidden']"
      >
        <div class="card">
          <div class="card-header msg_head">
            <div class="d-flex bd-highlight">
              <div class="img_cont">
                <img
                  :src="friend.profile_picture"
                  class="rounded-circle user_img"
                />
                <span class="online_icon" v-if="friend.loggedIn"></span>
                <span class="online_icon offline" v-else></span>
              </div>
              <div class="user_info">
                <span>Chat with {{ friend.name }}</span>
                <p>{{ messageCounter }} Messages</p>
              </div>
              <div class="video_cam">
                <span><i class="fas fa-video"></i></span>
                <span><i class="fas fa-phone"></i></span>
              </div>
            </div>
            <span id="action_menu_btn"><i class="fas fa-ellipsis-v"></i></span>
            <div class="action_menu">
              <ul>
                <li><i class="fas fa-user-circle"></i> View profile</li>
                <li><i class="fas fa-users"></i> Add to close friends</li>
                <li><i class="fas fa-plus"></i> Add to group</li>
                <li><i class="fas fa-ban"></i> Block</li>
              </ul>
            </div>
          </div>
          <div class="card-body msg_card_body">
            <div
              class="d-flex mb-4"
              :class="[
                message.author === authUser.uid
                  ? 'justify-content-end'
                  : 'justify-content-start',
              ]"
              v-for="(message, index) in messages"
              :key="index"
            >
              <div
                :class="[
                  message.author === authUser.uid
                    ? 'img_cont_msg_send'
                    : 'img_cont_msg',
                ]"
              >
                <img
                  :src="friend.profile_picture"
                  class="rounded-circle user_img_msg"
                />
              </div>
              <div
                :class="[
                  message.author === authUser.uid
                    ? 'msg_cotainer_send'
                    : 'msg_cotainer',
                ]"
              >
                {{ message.message }}
                <span class="msg_time">{{
                  message.createdAt | parseDate
                }}</span>
              </div>
            </div>
          </div>
          <div class="card-footer">
            <div class="input-group">
              <div class="input-group-append">
                <span class="input-group-text attach_btn"
                  ><i class="fas fa-paperclip"></i
                ></span>
              </div>
              <textarea
                v-model="message"
                name=""
                class="form-control type_msg"
                placeholder="Type your message..."
                @keyup.enter="saveMessage"
              ></textarea>
              <div class="input-group-append">
                <span class="input-group-text send_btn"
                  ><i class="fas fa-location-arrow"></i
                ></span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import firebase from "firebase";
export default {
  name: "PrivateChat",
  data() {
    return {
      users: [],
      authUser: {},
      friend: {},
      recieversId: null,
      message: null,
      messages: [],
      messageCounter: 0,
      isSelected: false,
    };
  },
  created() {
    firebase.auth().onAuthStateChanged((user) => {
      if (user) {
        this.authUser = user;

        var userStatusDatabaseRef = rtDb.ref("/users/" + user.uid);

        var isOfflineForDatabase = {
          uid: user.uid,
          name: user.displayName,
          loggedIn: false,
          last_changed: firebase.database.ServerValue.TIMESTAMP,
          profile_picture: user.photoURL,
        };

        var isOnlineForDatabase = {
          uid: user.uid,
          name: user.displayName,
          loggedIn: true,
          last_changed: firebase.database.ServerValue.TIMESTAMP,
          profile_picture: user.photoURL,
        };
        // Create a reference to the special '.info/connected' path in
        // Realtime Database. This path returns `true` when connected
        // and `false` when disconnected.
        rtDb.ref(".info/connected").on("value", function (snapshot) {
          // If we're not currently connected, don't do anything.
          if (snapshot.val() == false) {
            return;
          }

          // If we are currently connected, then use the 'onDisconnect()'
          // method to add a set which will only trigger once this
          // client has disconnected by closing the app,
          // losing internet, or any other means.
          userStatusDatabaseRef
            .onDisconnect()
            .set(isOfflineForDatabase)
            .then(function () {
              // The promise returned from .onDisconnect().set() will
              // resolve as soon as the server acknowledges the onDisconnect()
              // request, NOT once we've actually disconnected:
              // https://firebase.google.com/docs/reference/js/firebase.database.OnDisconnect
              // We can now safely set ourselves as 'online' knowing that the
              // server will mark us as offline once we lose connection.
              userStatusDatabaseRef.set(isOnlineForDatabase);
            });
        });
      } else {
        this.authUser = {};
      }
    });
    this.fetchAllUsers();
  },
  beforeRouteEnter(to, from, next) {
    // ...
    next((vm) => {
      firebase.auth().onAuthStateChanged((user) => {
        if (user) {
          next();
        } else {
          vm.$router.push("/login").catch((error) => {
            if (error.name != "NavigationDuplicated") {
              throw error;
            }
          });
        }
      });
    });
  },
  methods: {
    fetchAllUsers() {
      // Firestore
      // db.collection("users").onSnapshot((querySnapshot) => {
      //   let allUsers = [];
      //   querySnapshot.forEach((doc) => {
      //     if (
      //       this.authUser.uid !== {} &&
      //       doc.data().uid !== this.authUser.uid
      //     ) {
      //       allUsers.push(doc.data());
      //     }
      //   });
      //   this.users = allUsers;
      // });

      // Realtime Database
      rtDb.ref("users").on("value", (snapshot) => {
        let allUsers = [];
        snapshot.forEach((doc) => {
          if (this.authUser.uid !== {} && doc.val().uid !== this.authUser.uid) {
            allUsers.push(doc.val());
          }
        });
        this.users = allUsers;
      });
    },
    saveMessage() {
      db.collection("chat")
        .add({
          message: this.message,
          author: this.authUser.uid,
          reciever: this.recieversId,
          createdAt: new Date(),
        })
        .then(() => {
          this.scrollToBottom();
        });
      this.message = null;
    },
    readMessages(uid) {
      this.recieversId = uid;
      // Returns friend's data from Firestore
      // db.collection("users").onSnapshot((querySnapshot) => {
      //   querySnapshot.forEach((doc) => {
      //     if (doc.data().uid === uid) {
      //       this.friend = doc.data();
      //     }
      //   });
      // });

      // Returns friend's data from Realtime Database
      rtDb.ref("users/" + uid).once("value", (snapshot) => {
        this.friend = snapshot.val();
      });
      this.isSelected = true;
      this.messageCounter = 0;
      db.collection("chat")
        .orderBy("createdAt")
        .onSnapshot((querySnapshot) => {
          let allMessages = [];
          querySnapshot.forEach((doc) => {
            if (
              (doc.data().reciever === this.recieversId &&
                doc.data().author === this.authUser.uid) ||
              (doc.data().author === this.recieversId &&
                doc.data().reciever === this.authUser.uid)
            ) {
              allMessages.push(doc.data());
            }
          });
          this.messages = allMessages;
          this.messageCounter = allMessages.length;

          setTimeout(() => {
            this.scrollToBottom();
          }, 1000);
        });
    },
    scrollToBottom() {
      let box = document.querySelector(".msg_card_body");
      box.scrollTop = box.scrollHeight;
    },
  },
  filters: {
    parseDate(timestamp) {
      var date = new Date(timestamp * 1000);
      return date.getHours() + ":" + date.getMinutes();
    },
  },
  watch: {
    users: function () {
      if (this.users.length !== 0) {
        if (this.users[0].loggedIn === true) {
          this.friend.loggedIn = true;
        } else {
          this.friend.loggedIn = false;
        }
      }
    },
  },
};
</script>

<style scoped>
.chat {
  margin-top: auto;
  margin-bottom: auto;
}
.card {
  height: 500px;
  border-radius: 15px !important;
  background-color: rgba(0, 0, 0, 0.4) !important;
}
.contacts_body {
  padding: 0.75rem 0 !important;
  overflow-y: auto;
  white-space: nowrap;
}
.msg_card_body {
  overflow-y: auto;
}
.card-header {
  border-radius: 15px 15px 0 0 !important;
  border-bottom: 0 !important;
}
.card-footer {
  border-radius: 0 0 15px 15px !important;
  border-top: 0 !important;
}
.container {
  align-content: center;
}
.search {
  border-radius: 15px 0 0 15px !important;
  background-color: rgba(0, 0, 0, 0.3) !important;
  border: 0 !important;
  color: white !important;
}
.search:focus {
  box-shadow: none !important;
  outline: 0px !important;
}
.type_msg {
  background-color: rgba(0, 0, 0, 0.3) !important;
  border: 0 !important;
  color: white !important;
  height: 60px !important;
  overflow-y: auto;
}
.type_msg:focus {
  box-shadow: none !important;
  outline: 0px !important;
}
.attach_btn {
  border-radius: 15px 0 0 15px !important;
  background-color: rgba(0, 0, 0, 0.3) !important;
  border: 0 !important;
  color: white !important;
  cursor: pointer;
}
.send_btn {
  border-radius: 0 15px 15px 0 !important;
  background-color: rgba(0, 0, 0, 0.3) !important;
  border: 0 !important;
  color: white !important;
  cursor: pointer;
}
.search_btn {
  border-radius: 0 15px 15px 0 !important;
  background-color: rgba(0, 0, 0, 0.3) !important;
  border: 0 !important;
  color: white !important;
  cursor: pointer;
}
.contacts {
  list-style: none;
  padding: 0;
}
.contacts li {
  width: 100% !important;
  padding: 5px 10px;
  margin-bottom: 15px !important;
}
.active {
  background-color: rgba(0, 0, 0, 0.3);
}
.user_img {
  height: 70px;
  width: 70px;
  border: 1.5px solid #f5f6fa;
}
.user_img_msg {
  height: 40px;
  width: 40px;
  border: 1.5px solid #f5f6fa;
}
.img_cont {
  position: relative;
  height: 70px;
  width: 70px;
}
.img_cont_msg {
  height: 40px;
  width: 40px;
}
.img_cont_msg_send {
  display: none;
}
.online_icon {
  position: absolute;
  height: 15px;
  width: 15px;
  background-color: #4cd137;
  border-radius: 50%;
  bottom: 0.2em;
  right: 0.4em;
  border: 1.5px solid white;
}
.offline {
  background-color: #c23616 !important;
}
.user_info {
  text-align: left;
  margin-top: auto;
  margin-bottom: auto;
  margin-left: 15px;
}
.user_info span {
  font-size: 20px;
  color: white;
}
.user_info p {
  text-align: left;
  font-size: 10px;
  color: rgba(255, 255, 255, 0.6);
}
.video_cam {
  margin-left: 50px;
  margin-top: 5px;
}
.video_cam span {
  color: white;
  font-size: 20px;
  cursor: pointer;
  margin-right: 20px;
}
.msg_cotainer {
  margin-top: auto;
  margin-bottom: auto;
  margin-left: 10px;
  border-radius: 25px;
  background-color: #82ccdd;
  padding: 10px;
  position: relative;
}
.msg_cotainer_send {
  margin-top: auto;
  margin-bottom: auto;
  margin-right: 10px;
  border-radius: 25px;
  background-color: #78e08f;
  padding: 10px;
  position: relative;
}
.msg_time {
  position: absolute;
  left: 0;
  bottom: -15px;
  color: rgba(255, 255, 255, 0.5);
  font-size: 10px;
}
.msg_time_send {
  position: absolute;
  right: 0;
  bottom: -15px;
  color: rgba(255, 255, 255, 0.5);
  font-size: 10px;
}
.msg_head {
  position: relative;
}
#action_menu_btn {
  position: absolute;
  right: 10px;
  top: 10px;
  color: white;
  cursor: pointer;
  font-size: 20px;
}
.action_menu {
  z-index: 1;
  position: absolute;
  padding: 15px 0;
  background-color: rgba(0, 0, 0, 0.5);
  color: white;
  border-radius: 15px;
  top: 30px;
  right: 15px;
  display: none;
}
.action_menu ul {
  list-style: none;
  padding: 0;
  margin: 0;
}
.action_menu ul li {
  width: 100%;
  padding: 10px 15px;
  margin-bottom: 5px;
}
.action_menu ul li i {
  padding-right: 10px;
}
.action_menu ul li:hover {
  cursor: pointer;
  background-color: rgba(0, 0, 0, 0.2);
}
.visible {
  visibility: visible;
}
.hidden {
  visibility: hidden;
}
@media (max-width: 576px) {
  .contacts_card {
    margin-bottom: 15px !important;
  }
}
</style>
