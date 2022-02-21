<template>
  <div id="app">
    <div style="display: flex; flex-direction: column; align-content: center; align-items: center">
      Simple {{ title }}
      <p>Result : {{ dataFromSocket }}</p>
      <div style="display: flex">
        <label>input something</label>
        <input v-model="dataInput" style="width: 100%" />
        <button @click="handleSend">Send</button>
      </div>
    </div>
  </div>
</template>

<script>
import SockJS from "sockjs-client";
import Stomp from "webstomp-client";

export default {
  name: 'App',
  data () {
    return {
      title: 'Web Socket Testing',
      dataFromSocket: [],
      dataInput: '',
      connection: null
    }
  },
  methods: {

    // ================= INI WEBSOCKET =================

    handleSend () {
      if(this.title === 'Web Socket Testing') {
        this.connection.send(this.dataInput)
      } else {
        this.send()
      }
    },
    connectWithWebSocket () {
      // you can change the url for connect in below
      this.connection = new WebSocket("wss://demo.piesocket.com/v3/channel_1?api_key=oCdCMcMPQpbvNjUIzqtvF1d2X2okWpDQj4AwARJuAgtjhzKxVEjQU6IdCjwm&notify_self")

      this.connection.onmessage = (event) => {
        this.dataFromSocket = event.data
        console.log(event.data, 'data');
        console.log(event, 'event');
      }

      this.connection.onopen = function() {
        console.log("Successfully connected to the echo websocket server...")
      }
    },

    // ================= INI SOCKJS =================

    send() {
      if (this.stompClient && this.stompClient.connected) {
        this.stompClient.send("https://6.tcp.ngrok.io:15628/topic/messages", this.dataInput, {});
      }
    },
    connectWithSockJs() {
      this.title = 'Sock JS Testing'
      // you can change the url for connect in below
      this.socket = new SockJS('https://6.tcp.ngrok.io:15628/our-websocket')
      console.log(this.socket, 'socket')
      this.stompClient = Stomp.over(this.socket);
      this.stompClient.connect(
        {},
        frame => {
          this.connected = true;
          console.log(frame, 'frame');
          // you can change the url for subscribe in below
          this.stompClient.subscribe('https://6.tcp.ngrok.io:15628/topic/messages', tick => {
            console.log(tick, 'tick');
            this.dataFromSocket = tick.body
            // this.received_messages.push(JSON.parse(tick.body).content);
          });
        },
        error => {
          console.log(error, 'error');
          this.connected = false;
        }
      );
    },
  },
  mounted() {
    this.connectWithWebSocket()
    // this.connectWithSockJs();
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
