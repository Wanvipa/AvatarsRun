<template >
  <div class="" v-show="logOn">
    <input type="text" name="name" value="" v-model="name">
    <button type="button" name="button" @click="setNewAvatar">logOn</button>
  </div>
  <div id="app" v-show="!logOn">
    <div class="avatar" :style="{'background': avatar.color, 'top': avatar.y + 'px', 'left': avatar.x + 'px'}" v-for="avatar in avatars">
      {{avatar.name}}
      {{avatar.status}}
    </div>
    <input type="text" v-model="status" @keydown.enter="editStatus(status)">
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
export default {
  beforeDestroy () {},
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
    let r = Math.floor(Math.random() * 255)
    let g = Math.floor(Math.random() * 255)
    let b = Math.floor(Math.random() * 255)
    let x = Math.floor(Math.random() * 500)
    let y = 0
    return {
      avatars: [],
      myAvatar: {
        color: `rgb(${r}, ${g}, ${b})`,
        x,
        y,
        name: '',
        status: ''
      },
      logOn: true,
      name: '',
      status: ''
    }
  },
  // computed property for form validation state
  computed: {},
  // methods
  methods: {
    setNewAvatar: function () {
      this.myAvatar.name = this.name
      this.logOn = false
      this.addAvatar(this.myAvatar)
    },
    addAvatar: function (newAvatar) {
      let result = Avatars.push(newAvatar)
      this.myAvatar.id = result.key
    },
    editStatus: function (status) {
      var vm = this
      vm.status = ''
      firebase.database().ref('avatars/' + vm.myAvatar.id).update({
        status: status
      })
    },
    move () {
      console.log('move')
      let vm = this
      firebase.database().ref('avatars/' + vm.myAvatar.id).update({
        y: vm.myAvatar.y + 10
      })
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
  background-color: #69778a
}

#app {}

.avatar {
  position: absolute;
  width: 100px;
  height:100px;
  background: #F00;
}
</style>
