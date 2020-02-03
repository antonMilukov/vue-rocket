<template>
  <div>
    <h1>{{ intro }}</h1>

    <div>
      <p>
          <input id="username" type="text" v-model="username"> Username
      </p>

      <p>
          <input id="password" type="password" v-model="password"> Password
      </p>
      <p>
        <input type="button" v-on:click="auth" value="auth">
      </p>
    </div>

    <h3>Write a message</h3>
    <div>
      <p>
        <input id="new_message" type="text" v-model="newMessage"> New message
      </p>

      <p>
        <input type="button" v-on:click="sendMessage" value="send">
      </p>
    </div>


    <div v-if="roomId">
      <h3>Messages in room {{ roomId}}</h3>
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
      intro: "RocketVue component",
      websocketUrl: "wss://demo.rocket.chat/websocket",
      username: '',
      password: '',
      messages: [],
      newMessage: '',
      roomId: 'Drjw54ftqGa4antMW',
      authState: null
    };
  },
  mounted() {
    api = new RealTimeAPI(this.websocketUrl);
    api.keepAlive().subscribe();
    api.onMessage(message =>{
      this.messages.push(message);
    });
  },
  methods: {
    auth(){
      this.authState = api.login(this.username, this.password);
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
            "rid": this.roomId,
            "msg": this.newMessage
          }
        ]
      });
    }
  }
}
</script>