<template>
  <div>
    <div id="profile">
      <div id="profile-image-section">
        <div id="profile-image">
          <img v-bind:src="user.profilePic" />
        </div>
        <button v-show="isCurrentUserProfile && !showEditPic" v-on:click="showEditPic = !showEditPic">Edit</button>
        <h1>{{user.username}}</h1>
      </div>
      <div id="bio-section">
        <p>{{user.bio}}</p>
        <button v-show="isCurrentUserProfile && !showEditBio" v-on:click="showEditBio = !showEditBio">Edit</button>
        <div class="edit" v-show="showEditBio">
          <form class="editForm">
            <textarea class="edittext" v-model="newBio" placeholder="Edit Bio..."/>
            <button v-on:click.prevent="updateUser()">Submit</button>
            <button v-on:click.prevent="showEditBio = !showEditBio">Cancel</button>
          </form>
        </div>
        <div class="edit" v-show="showEditPic">
          <form class="editForm">
            <textarea v-model="newProfileUrl" class="edittext" placeholder="Edit Pic..."/>
            <button v-on:click.prevent="updateUser()">Submit</button>
            <button v-on:click.prevent="showEditPic = !showEditPic">Cancel</button>
          </form>
        </div>
      </div>
    </div>
  <Activity></Activity>
  </div>
</template>

<script>
import userService from '../services/UserService'
import Activity from './Activity.vue'
export default {
  components: { Activity },
  name: 'user-details',
  userId: 0,
  data() {
    return {
      showEditBio: false,
      showEditPic: false,
      isCurrentUserProfile: false,
      newBio: '',
      newProfileUrl: '',
      user:{
        id: 0,
        username: "",
        profilePic: "",
        bio: "No bio",
        authorities: "",
      }
    }
  },
  methods: {
    updateUser() {
      const updatedUser = { 
        id: this.user.id,
        username: this.user.username,
        profilePic: (this.newProfileUrl ? this.newProfileUrl : this.user.profilePic),
        bio: (this.newBio ? this.newBio : this.user.bio),
        authorities: this.authorities
        };
      userService.updateUser(updatedUser).then(() => {
          this.user = updatedUser
          this.$store.commit("SET_USER", updatedUser);
          this.profilePic = "";
          this.bio = "";
          this.showEditBio = false;
          this.showEditPic = false;
      })
      .catch((error) => {
                if(error.response) {
                    //We found the server and got a response back
                    alert("Something went wrong: " + error.response.statusText);
                } else if(error.request){
                    //We could not reach the server
                    alert("We could not reach the server");
                } else {
                    alert("Something went horribly wrong");
                }
            })
    }
  },
  created() {
    this.userId = this.$route.params.id

    userService.getUserById(this.userId).then((response) => {
      console.log(response.data)
      this.user.id = response.data.id;
      this.user.username = response.data.username;
      this.user.profilePic = response.data.profilePic;
      console.log(response.data.profilePic)
      if (response.data.bio) {
        this.user.bio = response.data.bio;
      }
      this.user.authorities = response.data.authorities[0].name;
    })

    this.isCurrentUserProfile = (parseInt(this.$route.params.id) === this.$store.state.user.id)
  }
}
</script>

<style scoped>

  #profile {
    display: flex;
  }

  #profile-image-section {
    text-align: center;
  }

  #bio-section {
    margin-top: 40px;
    margin-left: 30px;
  }

  #profile-image {
    margin: 15px;
    border: solid 2px #555597;
    border-radius: 50%;
    height: 150px;
    width: 150px;
    overflow: hidden;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  #profile-image img {
    width: 200px;
  }

  .edittext {
    padding: 7px;
    font-family: 'Montserrat', sans-serif;
    font-size: 14px;
    font-weight:500;
    border-width: 6px;
    border-color: #3b7793;
    background-color: #FFFFFF;
    color: #000000;
    border-style: solid;
    border-radius: 14px;
    box-shadow: 3px 3px 5px rgba(66,66,66,.75);
  }

  button {
    background-color: #5390b3;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin: 10px;
  }

  button:hover {
    background-color: #225979;
  }

  .editForm {
    display: flex;
    align-items: center;
  }

  .edit {
    margin: 10px;
  }
</style>