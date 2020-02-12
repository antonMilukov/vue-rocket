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
      <h3>Available Rooms - click will provoke connection </h3>
      <ul>
        <li v-for="(room, $index) in rooms" :key="$index" style="cursor: pointer;" v-on:click="connectToRoom($index)">{{ formatRoom(room) }}</li>
      </ul>
    </div>

    <div v-if="connect.room">
      <h3>Write a message</h3>
      <p>
        <input id="new_message" type="text" v-model="newMessage"> New message
      </p>

      <p>
        <input type="button" v-on:click="sendMessage" value="send">
      </p>
    </div>

    <div v-if="connect.room">
      <h3>Messages in room {{ formatRoom(connect.room) }}</h3>
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
        room: null
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
      return api.callMethod("rooms/get", [ { "$date": new Date().getDate() } ]);
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

            this.getRooms().subscribe(
                res => {
                    if (res.msg !== "result") return;
                    // eslint-disable-next-line no-console
                    console.log("getRooms", res);
                    this.rooms = res.result;
                }
            )

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
            "rid": this.connect.room._id,
            "msg": this.newMessage
          }
        ]
      });
      this.newMessage = '';
    },

    // connect to selected room by index
    connectToRoom(index){
      this.connect.room = this.rooms[index]

        // eslint-disable-next-line no-console
      console.log(this.connect.room);

      this.$nextTick(()=>{
          api.callMethod("openRoom", [ this.connect.room._id ]);
      });

    },

    // prepare result name of room
    formatRoom(entity){
      var name = '-';
      switch (entity.t) {
        case "d":
          name = "private";
          break;
        case "p":
          name = entity.fname;
          break;
        default:
          name = entity.name;
          break;
      }
      return name;
    }
  }
}
</script>