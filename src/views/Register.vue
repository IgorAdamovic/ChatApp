<template>
  <div class="container">
    <div class="d-flex justify-content-center h-100">
      <div class="card">
        <div class="card-header">
          <h3>Sign Up</h3>
        </div>
        <div class="card-body">
          <form v-if="!registered">
            <div class="input-group form-group">
              <div class="input-group-prepend">
                <span class="input-group-text"
                  ><i class="fas fa-envelope"></i
                ></span>
              </div>
              <input
                type="text"
                class="form-control"
                placeholder="email"
                v-model="email"
              />
            </div>
            <div class="input-group form-group">
              <div class="input-group-prepend">
                <span class="input-group-text"><i class="fas fa-key"></i></span>
              </div>
              <input
                type="password"
                class="form-control"
                placeholder="password"
                v-model="password"
              />
            </div>
            <div class="form-group">
              <input
                type="submit"
                value="Register"
                class="btn float-right login_btn"
                @click.prevent="register"
              />
            </div>
          </form>
          <form v-else>
            <div class="input-group form-group">
              <div class="input-group-prepend">
                <span class="input-group-text"
                  ><i class="fas fa-user"></i
                ></span>
              </div>
              <input
                type="text"
                class="form-control"
                placeholder="username"
                v-model="username"
              />
            </div>
            <div class="input-group form-group" v-if="!image">
              <div class="input-group-prepend">
                <span class="input-group-text"
                  ><i class="fas fa-grimace"></i
                ></span>
              </div>
              <input type="file" class="form-control" @change="selectImage" />
            </div>
            <div v-else>
              <img :src="image" />
              <input
                type="button"
                class="btn removeImage"
                value="X"
                @click="removeImage"
              />
            </div>
            <div class="form-group">
              <input
                type="submit"
                value="Add"
                class="btn float-right login_btn"
                @click.prevent="addUsername"
              />
            </div>
          </form>
        </div>
        <div class="card-footer">
          <div class="d-flex justify-content-center links">
            Already have an account?<a href="/login">Sign In</a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import firebase from "firebase";

export default {
  data() {
    return {
      regUser: {},
      email: "",
      password: "",
      username: "",
      image: "",
      registered: false,
    };
  },
  methods: {
    register() {
      firebase
        .auth()
        .createUserWithEmailAndPassword(this.email, this.password)
        .then((userCredential) => {
          // Signed in
          var user = userCredential.user;
          // ...
          this.regUser = user;
          this.registered = true;
          //this.$router.push("/");
        })
        .catch((error) => {
          var errorCode = error.code;
          var errorMessage = error.message;
          // ..
        });
    },
    addUsername() {
      var user = firebase.auth().currentUser;
      var storageRef = storage.ref("pictures/" + this.regUser.uid + ".jpg");
      storageRef.getDownloadURL().then((url) => {
        user
          .updateProfile({
            displayName: this.username,
            photoURL: url,
          })
          .then(function () {
            // Update successful.
            // db.collection("users").doc(user.uid).set({
            //   uid: user.uid,
            //   name: user.displayName,
            //   loggedIn: false,
            //   last_changed: firebase.database.ServerValue.TIMESTAMP
            // });
            rtDb.ref("users/" + user.uid).set({
              uid: user.uid,
              name: user.displayName,
              loggedIn: false,
              last_changed: firebase.database.ServerValue.TIMESTAMP, 
              profile_picture: user.photoURL
            });
          })
          .catch(function (error) {
            // An error happened.
          });
      });
      this.$router.push("/login");
    },
    selectImage(e) {
      var files = e.target.files || e.dataTransfer.files;
      if (!files.length) return;
      this.createImage(files[0]);
      // Upload the file
      var file = e.target.files[0];
      var storageRef = storage.ref("pictures/" + this.regUser.uid + ".jpg");
      storageRef.put(file);
    },
    createImage(file) {
      var reader = new FileReader();
      var vm = this;

      reader.onload = (e) => {
        vm.image = e.target.result;
      };
      reader.readAsDataURL(file);
    },
    removeImage() {
      var storageRef = storage.ref("pictures/" + this.regUser.uid + ".jpg");
      // Delete the file
      storageRef
        .delete()
        .then(() => {
          // File deleted successfully
        })
        .catch((error) => {
          // Uh-oh, an error occurred!
        });
      this.image = "";
    },
  },
};
</script>

<style scoped>
html,
body {
  background-size: cover;
  background-repeat: no-repeat;
  height: 100%;
}

.container {
  height: 100%;
  align-content: center;
}

.card {
  margin-top: auto;
  margin-bottom: auto;
  width: 400px;
  background-color: rgba(0, 0, 0, 0.5) !important;
}

.social_icon span {
  font-size: 60px;
  margin-left: 10px;
  color: #ffc312;
}

.social_icon span:hover {
  color: white;
  cursor: pointer;
}

.card-header h3 {
  color: white;
}

.social_icon {
  position: absolute;
  right: 20px;
  top: -45px;
}

.input-group-prepend span {
  width: 50px;
  background-color: #ffc312;
  color: black;
  border: 0 !important;
}

input:focus {
  outline: 0 0 0 0 !important;
  box-shadow: 0 0 0 0 !important;
}

img {
  width: 100%;
  height: auto;
  margin-bottom: 20px;
}

.removeImage {
  padding: 2px 8px;
  margin: 5px;
  position: absolute;
  right: 24px;
  background-color: #ffc312;
  border-radius: 50%;
  outline: 0 0 0 0 !important;
  box-shadow: 0 0 0 0 !important;
}

.removeImage:hover {
  color: black;
  background-color: white;
}

.remember {
  color: white;
}

.remember input {
  width: 20px;
  height: 20px;
  margin-left: 15px;
  margin-right: 5px;
}

.login_btn {
  color: black;
  background-color: #ffc312;
  width: 100px;
}

.login_btn:hover {
  color: black;
  background-color: white;
}

.links {
  color: white;
}

.links a {
  margin-left: 4px;
}
</style>
