<template >
  <div id="app" >
    <div class="avatar" :style="{'background': avatar.color, 'top': avatar.y + 'px', 'left': avatar.x + 'px'}" v-for="avatar in avatars">
    </div>
  </div>
</template>
<script>
/*global alert */
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
  beforeDestroy () {
    alert('adasdads')
  },
  ready () {
    window.addEventListener('keydown', this.keyDown)
    let vm = this
    vm.addAvatar(vm.myAvatar)
    Avatars.on('child_added', function (snapshot) {
      var item = snapshot.val()
      item.id = snapshot.key
      vm.avatars.push(item)
    })
    Avatars.on('child_changed', function (snapshot) {
      var id = snapshot.key
      var avatar = vm.avatars.find(avatar => avatar.id === id)
      avatar.x = snapshot.val().x
      avatar.y = snapshot.val().y
      if (vm.myAvatar.id === id) {
        vm.myAvatar.x = snapshot.val().x
        vm.myAvatar.y = snapshot.val().y
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
        y
      }
    }
  },
  // computed property for form validation state
  computed: {
  },
  // methods
  methods: {
    addAvatar: function (newAvatar) {
      let result = Avatars.push(newAvatar)
      this.myAvatar.id = result.key
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
      console.log(event)
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
#app {
}
.avatar {
  position: absolute;
  width: 10px;
  height: 10px;
  background: #F00;
}
</style>
