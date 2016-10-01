<template >
  <div class="logOn" v-show="logOn">
    <div class="ui left corner labeled input">
      <input type="text" placeholder="Input You Name" v-model="name">
      <div class="ui left corner label">
        <i class="asterisk icon"></i>
      </div>
    </div>
    <div class="ui labeled input">
      <div class="ui label">
        http://
      </div>
      <input type="text" placeholder="Url: picture" v-model="picture">
    </div>
    <button class="ui basic button" @click="setNewAvatar">
      <i class="icon user"></i>logOn
    </button>
  </div>
  <div id="app" v-show="!logOn">
    <div class="avatar" :style="{'top': avatar.y + 'px', 'left': avatar.x + 'px'}" v-for="avatar in avatars">
      <div class="ui centered card">
        <div class="image">
          <img :src="avatar.picture" v-show="avatar.picture">
        </div>
        <div class="content">
          <i class="right floated remove icon" @click="removeAvatar"></i>
          <a class="header">{{avatar.name}}</a>
          <div class="description">
            {{avatar.status}}
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="status" v-show="!logOn">
    <input type="text" v-model="status">
    <button type="button" name="button" @click="editStatus()">Edit Status</button>
  </div>
  </div>
</template>
<script>
// /*global alert */
import firebase from 'firebase'
var config = {
  apiKey: 'AIzaSyBvZQ5jYSY7hECHidvWd4F3btGgRkhOYdQ',
  authDomain: 'todos-50735.firebaseapp.com',
  databaseURL: 'https://todos-50735.firebaseio.com',
  storageBucket: 'todos-50735.appspot.com',
  messagingSenderId: '514818084106'
}
firebase.initializeApp(config)

var Avatars = firebase.database().ref('avatars')
window.onbeforeunload = function () {
  firebase.database().ref('avatars/' + myId).remove()
}
var myId = ''
export default {
  ready () {
    window.addEventListener('keydown', this.keyDown)
    let vm = this
      // then add new Avatar
    Avatars.on('child_added', function (snapshot) {
      var item = snapshot.val()
      item.id = snapshot.key
      vm.avatars.push(item)
    })
      // then Avatars movement
    Avatars.on('child_changed', function (snapshot) {
      var id = snapshot.key
      var avatar = vm.avatars.find(avatar => avatar.id === id)
      avatar.x = snapshot.val().x
      avatar.y = snapshot.val().y
      avatar.status = snapshot.val().status
      if (vm.myAvatar.id === id) {
        vm.myAvatar.x = snapshot.val().x
        vm.myAvatar.y = snapshot.val().y
        vm.myAvatar.status = snapshot.val().status
      }
    })
    Avatars.on('child_removed', function (snapshot) {
      var id = snapshot.key
      vm.avatars.$remove(vm.avatars.find(avatar => avatar.id === id))
    })
  },
  data () {
    // let r = Math.floor(Math.random() * 255)
    // let g = Math.floor(Math.random() * 255)
    // let b = Math.floor(Math.random() * 255)
    let x = Math.floor(Math.random() * 500)
    let y = 0
    return {
      avatars: [],
      myAvatar: {
        // color: `rgb(${r}, ${g}, ${b})`,
        x,
        y,
        name: '',
        picture: '',
        status: ''
      },
      logOn: true,
      name: '',
      picture: '',
      status: ''
    }
  },
  // computed property for form validation state
  computed: {},
  // methods
  methods: {
    setNewAvatar: function () {
      this.myAvatar.name = this.name
      this.myAvatar.picture = this.picture
      this.logOn = false
      this.addAvatar(this.myAvatar)
    },
    addAvatar: function (newAvatar) {
      let result = Avatars.push(newAvatar)
      this.myAvatar.id = result.key
      myId = this.myAvatar.id
    },
    removeAvatar: function () {
      firebase.database().ref('avatars/' + this.myAvatar.id).remove()
      this.myAvatar = {}
    },
    editStatus: function () {
      var vm = this
      firebase.database().ref('avatars/' + vm.myAvatar.id).update({
        status: vm.status
      })
      vm.status = ''
    },
    // click evter
    keyDown (event) {
      let vm = this
      if (event.key === 'ArrowUp') {
        firebase.database().ref('avatars/' + vm.myAvatar.id).update({
          y: vm.myAvatar.y - 10
        })
      }
      if (event.key === 'ArrowDown') {
        firebase.database().ref('avatars/' + vm.myAvatar.id).update({
          y: vm.myAvatar.y + 10
        })
      }
      if (event.key === 'ArrowLeft') {
        firebase.database().ref('avatars/' + vm.myAvatar.id).update({
          x: vm.myAvatar.x - 10
        })
      }
      if (event.key === 'ArrowRight') {
        firebase.database().ref('avatars/' + vm.myAvatar.id).update({
          x: vm.myAvatar.x + 10
        })
      }
    }
  }
}
</script>

<style>
body {
  background-color: #69778a;
  width: 700px;
  overflow: hidden;
}

.logOn {
  margin-top: 500px;
  margin-left: 500px;
}

.status {
  margin-top: 500px;
  margin-left: 500px;
}

.avatar {
  position: absolute;
  width: 200px;
  /*height:100px;*/
  /*background: #F00;*/
}
</style>
