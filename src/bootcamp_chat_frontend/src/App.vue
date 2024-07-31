<script lang="ts">
import { ref } from 'vue';
import { bootcamp_chat_backend, canisterId, createActor } from '../../declarations/bootcamp_chat_backend';
import { AuthClient } from '@dfinity/auth-client';
import { HttpAgent } from '@dfinity/agent';
import type { Identity } from '@dfinity/agent';
import { Principal } from '@dfinity/principal';

export default {
  data() {
    return {
      newChat: "",
      chats: [] as string[][],
      identity: undefined as undefined | Identity,
      principal: undefined as undefined | Principal,
      targetPrincipal: "",
      userData: undefined as undefined | UserData
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
      await backend.add_chat_msg(this.newChat, targetPrincipal)
      await this.pobierzChaty()
    },
    async pobierzChaty() {
      const { identity, principal } = this.isUserLogged()
      const targetPrincipal = this.validateTargetPrincipal()

      const chatPath = [targetPrincipal, identity.getPrincipal()].sort()
      this.chats = await bootcamp_chat_backend.get_chat(chatPath)
    },
    async login() {
      const authClient = await AuthClient.create();
      await authClient.login({
        identityProvider: "http://be2us-64aaa-aaaaa-qaabq-cai.localhost:4943/",
        onSuccess: async () => {
          const identity = authClient.getIdentity();
          const principal = identity.getPrincipal();
          this.principal = principal;
          this.identity = identity;
          console.log("Zalogowano", this.principal)
          const maybeUserData = await bootcamp_chat_backend.get_user(principal)
          if (maybeUserData.length === 0) {
            this.userData = undefined
          } else {
            this.userData = maybeUserData[0]
          }
        }
      })
    },
    async logout() {
    const authClient = await AuthClient.create();
    await authClient.logout();
    this.principal = undefined;
    this.identity = undefined;
    this.chats = [];
    this.userData = undefined;

  },


  },
}
</script>

<template>
  <main>
    <img src="/logo2.svg" alt="DFINITY logo" />
    <br />
    <br />
    {{ principal }}
    <button v-if="!principal"  @click="login">login</button>
    <button v-if="principal"  @click="logout">logout</button>
    <div>
      <input v-model="targetPrincipal" /><button @click="pobierzChaty">pobierz chat</button>
    </div>
    <div v-if="principal">
      <div>
        <div v-for="chat in chats[0]">
          {{ chat }}
        </div>
      </div>
    </div>
    <div>
      <textarea v-model="newChat"></textarea><button @click="dodajChatMSG">Dodaj notatke</button>
    </div>
  </main>
</template>
