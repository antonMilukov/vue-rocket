<!--@todo: auth, sync with interval, -->
<template>
  <div>
    <h1>{{ intro }}</h1>

    <div>
      <p>
          <input id="username" type="text" v-model="username" autocomplete="username" name="username"> Username
      </p>

      <p>
          <input id="password" type="password" v-model="password" autocomplete="password" name="password"> Password
      </p>
      <p>
        <input type="button" v-on:click="authAndGetRooms" value="auth">
      </p>
    </div>

    <div v-if="connect.userId && connect.authToken">
      <h3>Available Rooms</h3>
      <ul>
        <li v-for="(room, $index) in rooms" :key="$index">{{ room }}</li>
      </ul>
    </div>

    <div v-if="rooms.length > 0">
      <h3>Connect to room</h3>
      <p>
        <input id="roomId" type="text" v-model="connect.roomId"> Id
      </p>
      <p>
        <input type="button" v-on:click="connectToRoom" value="connect">
      </p>
    </div>

    <div v-if="connect.roomId">
      <h3>Write a message</h3>
      <p>
        <input id="new_message" type="text" v-model="newMessage"> New message
      </p>

      <p>
        <input type="button" v-on:click="sendMessage" value="send">
      </p>
    </div>

    <div v-if="connect.roomId">
      <h3>Messages in room {{ connect.roomId}}</h3>
      <ul>
        <li v-for="(message, $index) in messages" :key="$index">{{ message }}</li>
      </ul>
    </div>

  </div>
</template>

<script>
import {RealTimeAPI} from 'rocket.chat.realtime.api.rxjs'
let api = null;
export default {
  name: 'VueRocket',
  data: ()=>{
    return {
      connect: {
        authToken: null,
        userId: null,
        roomId: null
      },
      rooms: [],
      intro: "RocketVue component",
      websocketUrl: "wss://open.rocket.chat/websocket",
      username: '',
      password: '',
      messages: [],
      newMessage: '',
      authState: 0
    };
  },
  mounted() {
    api = new RealTimeAPI(this.websocketUrl);
    api.connectToServer();
    api.keepAlive().subscribe();
    api.onMessage(message =>{
      this.messages.push(message);
    });
  },
  methods: {
    auth(){
      return api.login(this.username, this.password);
    },
    getRooms(){
      //
    },
    authAndGetRooms(){
      this.auth().subscribe(
        res => {
          if (res.msg !== "result") return;

          // eslint-disable-next-line no-console
          console.log(res);
          if ('result' in res){
            if ('token' in res.result){
              this.connect.authToken = res.result.token;
            }

            if ('id' in res.result){
              this.connect.userId = res.result.id;
            }


          }
        }
      );
    },

    sendMessage(){
      let id = new Date().getDate();
      api.sendMessage({
        "msg": "method",
        "method": "sendMessage",
        "id": id,
        "params": [
          {
            "_id": id,
            "rid": this.connect.roomId,
            "msg": this.newMessage
          }
        ]
      });
      this.newMessage = '';
    },

    connectToRoom(){
      api.sendMessage({
        "msg": "method",
        "method": "openRoom",
        "id": this.connect.roomId,
        "params": [
          "roomId"
        ]
      });
    }
  }
}
</script>