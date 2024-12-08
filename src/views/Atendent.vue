<template>
  <div id="general">
    <div id="left">
      <img class="logo" src="../assets/logo.png" alt="">
      <div class="row space-between">
        <h2 class="green">Serviços</h2>
        <div class="row">
          <img class="icon" src="../assets/user.svg" alt="">
          <p class="text green"> {{ total }}</p>
        </div>
      </div>
      <div id="servicesList">
        <div v-for="(sector, key) in queueData">
          <router-link :to="{ params: { service: sector.code } }" id="link">
            <div class="row space-between paddingLR" v-if="sector.code != this.$route.params.service">
              <p class="green">{{ sector.name }}</p>
              <div class="row">
                <img class="icon" src="../assets/user.svg" alt="">
                <p> {{ sector.clients.length }}</p>
              </div>
            </div>
            <div class="row space-between paddingLR selected" v-if="sector.code == this.$route.params.service">
              <p class="paddingLeft green">{{ sector.name }}</p>
              <div class="row">
                <img class="icon" src="../assets/user.svg" alt="">
                <p class="green">{{ sector.clients.length }}</p>
              </div>
            </div>
          </router-link>
          <hr v-if="key != 'biblioteca'" class="servicesList">
        </div>
      </div>

      <button id="finishServiceButton" @click="finishService" v-if="status == 'open'">Encerrar serviço</button>
      <button id="startServiceButton" @click="startService"
        v-if="status == 'closed' && serviceQueue.length == 0">Iniciar serviço</button>
      <button id="serviceButton" @click="startService" v-if="status == 'closed' && serviceQueue.length != 0">Termine de
        atender os clientes</button>
    </div>
    <div id="right">
      <div id="ticketForm">
        <div id="upperPartTicket" class="centerPadding3">
          <p class="grey font-wheight-600 text">Senha atual</p>
          <p class="currentTicketNumber" v-if="this.$route.params.service">
            {{ lastTicket }}
          </p>
          <h2 class="green service">{{ formattedRouteName }}</h2>
        </div>

        <div id="divider">
          <span class="dot"></span>
          <hr>
          <span class="dot"></span>
        </div>

        <div id="bottomPartTicket" class="centerPadding4">
          <div id="minorButtons">
            <button id="cancelButton" class="minorActionButton font-wheight-700">Cancelar</button>
            <button class="minorActionButton font-wheight-700">Chamar denovo</button>
          </div>
          <button id="nextTicketButton" class="font-wheight-700" v-if="this.$route.params.service"
            @click="queueStore.callNextTicket(this.$route.params.service)" :disabled="!serviceQueue.length">
            Próxima senha
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { useQueueStore } from '@/stores/queueStore';

export default {
  data() {
    return {
      queueStore: useQueueStore(),
    }
  },

  mounted() {
    if (!this.queueStore.connected) this.queueStore.connect();
    const route = this.$route.params.service;

    if (route) {
      this.queueStore.subscribeToClients(route);
      this.queueStore.fetchQueueDataPerService(route)
    }
  },

  beforeUnmount() {
    this.queueStore.disconnect();
  },

  created() {
    this.queueStore.fetchQueueDataPerService(this.$route.params.service);
    this.queueStore.fetchQueueData();
  },

  computed: {
    serviceQueue() {
      return this.queueStore.getClients
    },

    total() {
      const uniqueClients = [];
      for (const sector in this.queueData) {
        for (const client of this.queueData[sector].clients) {
          uniqueClients.push(client);
        }
      }
      return uniqueClients.length;
    },

    lastTicket() {      
      return `${this.serviceData.key}${this.queueStore.getLastTicketCalled.toString().padStart(2, "0")}`;
    },

    formattedRouteName() {
      return this.serviceData.name
    },

    status() {
      return this.queueStore.getStatus
    },

    queueData() {
      return this.queueStore.getData
    },

    serviceData() {      
      return this.queueStore.getServiceData
    }
  },

  methods: {
    finishService() {
      this.queueStore.finishService(this.$route.params.service);
    },

    startService() {
      this.queueStore.startService(this.$route.params.service);
    }
  },

  watch: {
    serviceQueue() {
      this.queueStore.fetchQueueDataPerService(this.$route.params.service);
    },

    queueData() {
      this.queueStore.fetchQueueData();
    },

    '$route.params.service': {
      immediate: true,
      handler(newRoute) {
        this.queueStore.subscribeToClients(newRoute);
        this.queueStore.fetchQueueDataPerService(newRoute);
      }
    }
  },
}
</script>

<style scoped>
@font-face {
  font-family: "Karla";
  src: url(https://fonts.gstatic.com/s/karla/v31/qkB9XvYC6trAT55ZBi1ueQVIjQTD-JrIH2G7nytkHRyQ8p4wUjm6bnEr.woff2) format('woff2');
}

* {
  font-family: "Karla", sans-serif;
}

body,
html {
  margin: 0;
  padding: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

p {
  font-size: 18px;
}

#general {
  background-color: #307e69 !important;
  display: flex;
  flex-direction: row;
  height: 100vh;
  width: 100vw;
  margin: 0;
  padding: 0;
}

#left {
  width: 25%;
  background-color: #E7E8E3;
  padding: 2%;
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  align-content: center;
}

#right {
  width: 75%;
  background-color: #307E69;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

img.logo {
  max-width: 50%;
  max-height: 20%;
  margin-left: 25%;
}

.row {
  display: flex;
  flex-direction: row;
  align-items: center;
}

.space-between {
  display: flex;
  justify-content: space-between;
}

.icon {
  max-width: 20px;
  max-height: 20px;
  margin-right: 8px;
}

.text {
  font-size: 22px;
}

.grey {
  color: #949983;
}

.green {
  color: #307E69;
}

.font-wheight-600 {
  font-weight: 600;
}

.font-wheight-700 {
  font-weight: 700;
}

#ticketForm {
  background-color: #ffffff;
  border-radius: 1.5em;
  margin: 3em;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.currentTicketNumber {
  color: #307e69;
  font-size: 80px;
  font-weight: 900;
  padding: 0;
  margin: 0;
}

#divider {
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  align-items: center;
  padding-top: 2%;
  padding-bottom: 6%;
  margin-left: -24px;
  margin-right: -24px;
}

.dot {
  height: 25px;
  width: 25px;
  background-color: #307e69;
  border-radius: 50%;
  display: inline-block;
  margin-left: 10px;
  margin-right: 10px;
}

hr {
  border: none;
  border-top: 6px dotted #307E69;
  width: 30vw;
  margin: 0;
}

.centerPadding3 {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-around;
  padding: 8%;
  width: 85%;
}

.centerPadding4 {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-around;
  padding-left: 8%;
  padding-right: 8%;
  padding-bottom: 8%;
  width: 85%;
}

#minorButtons {
  width: 100%;
  display: flex;
  justify-content: space-between;
}

#nextTicketButton {
  width: 100%;
  padding: 4%;
  border: none;
  border-radius: 10px;
  background-color: #307E69;
  color: white;
  margin-top: 5%;
  font-size: 18px;
}

#nextTicketButton:disabled {
  width: 100%;
  padding: 4%;
  border: none;
  border-radius: 10px;
  background-color: #307E6980;
  color: white;
  margin-top: 5%;
  font-size: 18px;
  cursor: not-allowed;
}

#cancelButton {
  color: red;
}

#servicesList {
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  background-color: white;
  border-radius: 10px;
}


hr.servicesList {
  border: none;
  border-top: 1px solid #E7E8E3;
  width: 100%;
}

.paddingLR {
  padding-left: 2em;
  padding-right: 2em;
}

#finishServiceButton {
  padding: 4%;
  width: 100%;
  border: none;
  border-radius: 100px;
  background-color: red;
  color: white;
  margin-top: 8%;
  font-weight: 700;
  font-size: 18px;
}

#startServiceButton {
  padding: 4%;
  width: 100%;
  border: none;
  border-radius: 100px;
  background-color: #307E69;
  color: white;
  margin-top: 8%;
  font-weight: 700;
  font-size: 18px;
}

#serviceButton {
  padding: 4%;
  width: 100%;
  border: none;
  border-radius: 100px;
  color: black;
  margin-top: 8%;
  font-weight: 700;
  font-size: 18px;
}

.minorActionButton {
  background-color: #E7E8E3;
  border-radius: 10px;
  border: none;
  color: #307E69;
  width: 48%;
  padding: 4%;
  font-size: 18px;
}

.service {
  text-align: center;
}

#link {
  text-decoration: none;
  color: #307E69;
}

.selected {
  background-color: #f5f5f5;
  border-radius: 10px;
}
</style>