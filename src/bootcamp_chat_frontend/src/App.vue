<script lang="ts">
import { ref } from 'vue';
import { bootcamp_chat_backend, canisterId, createActor } from '../../declarations/bootcamp_chat_backend';
import { AuthClient } from '@dfinity/auth-client';
import { HttpAgent } from '@dfinity/agent';
import type { Identity } from '@dfinity/agent';
import { Principal } from '@dfinity/principal';
import type { UserData } from '../../declarations/bootcamp_chat_backend/bootcamp_chat_backend.did';

export default {
  data() {
    return {
      newChat: "",
      chats: [] as string[][],
      identity: undefined as undefined | Identity,
      principal: undefined as undefined | Principal,
      targetPrincipal: "",
      userData: undefined as undefined | UserData,
      newUsername: "",
      allUsers: [] as [Principal, UserData][]
    }
  },
  methods: {
    isUserLogged() {
      if (!this.identity || !this.principal || this.principal === Principal.anonymous()) {
        throw new Error("PLZ log in")
      }
      return {
        identity: this.identity,
        principal: this.principal
      }
    },
    validateTargetPrincipal() {
      const cleanTargetPrincipal = this.targetPrincipal.trim();
      if (cleanTargetPrincipal === "") {
        throw new Error("No principal")
      }
      const targetPrincipal = Principal.fromText(cleanTargetPrincipal)
      if (!targetPrincipal || targetPrincipal === Principal.anonymous()) {
        throw new Error("Wrong target")
      }
      return targetPrincipal
    },
    getAuthClient() {
      this.isUserLogged()
      return createActor(canisterId, {
        agentOptions: {
          identity: this.identity
        }
      });
    },
    async dodajChatMSG() {
      const targetPrincipal = this.validateTargetPrincipal()
      const backend = this.getAuthClient();
      console.log("Before calling add_chat_msg")
      await backend.add_chat_msg(this.newChat, targetPrincipal)
      console.log("After calling add_chat_msg")
      console.log("Before calling pobierzChaty")
      await this.pobierzChaty()
      console.log("After calling pobierzChaty")
    },
    async pobierzChaty() {
      const { identity, principal } = this.isUserLogged()
      const targetPrincipal = this.validateTargetPrincipal()

      const chatPath = [targetPrincipal, identity.getPrincipal()].sort()
      console.log("Before calling get_chat")
      this.chats = await bootcamp_chat_backend.get_chat(chatPath)
      console.log("After calling get_chat", this.chats)
    },
    async login() {
      console.log("Before calling AuthClient.create")
      const authClient = await AuthClient.create();
      console.log("After calling AuthClient.create")
      console.log("Before calling login")
      await authClient.login({
        //identityProvider: "http://be2us-64aaa-aaaaa-qaabq-cai.localhost:4943/",
        identityProvider: "http://b77ix-eeaaa-aaaaa-qaada-cai.localhost:4943/",
        
        onSuccess: async () => {
          console.log("Before calling getIdentity in onSuccess")
          const identity = authClient.getIdentity();
          console.log("After calling getIdentity in onSuccess") 
          console.log("Before calling getPrincipal in onSuccess")
          const principal = identity.getPrincipal();
          console.log("After calling getPrincipal in onSuccess")
          this.principal = principal;
          this.identity = identity;
          console.log("Zalogowano", this.principal)
          console.log("Before calling getUserData in onSuccess")
          await this.getUserData()
          console.log("After calling getUserData in onSuccess") 
          console.log("Before calling getAllUsers in onSuccess")  
          await this.getAllUsers()
          console.log("After calling getAllUsers in onSuccess")
        }
      })
      console.log("After calling login")
    },
    async logout () {
      console.log("Before calling AuthClient.create")
      const authClient = await AuthClient.create();
      console.log("After calling AuthClient.create")
      console.log("Before calling logout")
      await authClient.logout()
      console.log("After calling logout")
      this.identity = undefined;
      this.principal = undefined;
      this.chats = [];
      this.userData = undefined
    },
    async registerUsername() {
      const trimedUsername = this.newUsername.trim();
      const backend = this.getAuthClient();
      await backend.register(trimedUsername)
      await this.getUserData()
      await this.getAllUsers()
    },
    async getUserData() {
      const {principal} = this.isUserLogged()
      const maybeUserData = await bootcamp_chat_backend.get_user(principal as Principal)
          if (maybeUserData.length === 0) {
            this.userData = undefined
          } else {
            this.userData = maybeUserData[0]
          }
        console.log("User data", this.userData)
    },
    async getAllUsers(){
      this.allUsers = await bootcamp_chat_backend.get_users()
    }
  },
}
</script>

<template>
  <main>
    <button v-if="!principal" @click="login">login</button>
    <button v-if="principal" @click="logout">logout</button>
    <div v-if="principal && !userData">
      <input v-model="newUsername" placeholder="nick"/> <button @click="registerUsername">register</button>
    </div>
    <div v-if="principal && userData">
      {{ userData.nickname }}
      <div v-if="allUsers">
        <select v-model="targetPrincipal">
          <option disabled value="">Please select one</option>
          <option v-for="[userPrincipal, userData] in allUsers" :value="userPrincipal.toText()">{{ userData.nickname }}</option>
        </select>
      </div>
      <div>
        <div v-for="chat in chats[0]">
          {{ chat }}
              </div>
    </div>
    <div>
      <textarea v-model="newChat" placeholder="wiadomosc"></textarea><button @click="dodajChatMSG">Dodaj notatke</button>
</div>
    </div>
  </main>
</template>
