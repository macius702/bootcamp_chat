<script lang="ts">
import { ref } from 'vue';
import { bootcamp_chat_backend } from '../../declarations/bootcamp_chat_backend/index';



export default {
  data() {
    return {
      newNote:"",
      notes : [] as string[]
    };
  },
  methods:
  {
    async dodajNotatke(){
      console.log("dodajNotatke");
      await bootcamp_chat_backend.add_note( this.newNote );
    },
    async pobierzNotatki(){
      console.log("pobierzNotatki");
      this.notes = await bootcamp_chat_backend.get_notes();

    },

    mounted()    {  this.pobierzNotatki();}
  }
};
</script>

<template>
  <main>
    <img src="/logo2.svg" alt="DFINITY logo" />
    <br />
    <br />
    <div>
    {{ notes }}
  </div>
  <div>
    <textarea v-model="newNote" placeholder="Type your message here">Dodaj notatkę</textarea>
  </div>
  </main>
</template>
