<template>
    <button @click="selectScreen('inbox')"
            :disabled="selectedScreen == 'inbox'">Inbox</button>
    <button @click="selectScreen('archive')"
            :disabled="selectedScreen == 'archive'">Archived</button>
    <BulkActionBar :emails="unarcheivedEmails" />
    <table class="mail-table">
      <tbody>
        <tr 
        :class="['clickable',email.read ? 'read':'']"
        v-for="email in unarcheivedEmails"
        :key="email.id">
          <td>
            <input type="checkbox" 
                @click="emailSelection.toggle(email)"
                :checked="emailSelection.emails.has(email)" />
          </td>
          <td @click="openEmail(email)">
            {{email.from}}
          </td>
          <td @click="openEmail(email)">
            <p><strong>{{email.subject}}</strong> - {{email.body}}</p>
          </td>
          <td class="date" @click="openEmail(email)">
            {{format(new Date(email.sentAt), 'MMM do yyyy')}}
          </td>
          <td><button :disabled="email.archived==true" class="button" @click="archiveEmail(email)">Archived</button></td>
        </tr>
      </tbody>
    </table>
    <ModalView v-if="openedEmail" @closeModal = "openedEmail = null">
      <MailView :email= "openedEmail" @changeEmail="changeEmail"/>
    </ModalView>
</template>

<script>
import useEmailSelection from '../composition/useEmailSelection';
import MailView from "./MailView.vue"
import ModalView from "./ModalView.vue"
import BulkActionBar from './BulkActionBar.vue'
import { format } from 'date-fns';
import axios from "axios";
import {ref} from 'vue'
export default {
  components:{
    MailView,
    ModalView,
    BulkActionBar
  },
  data(){
    //await new Promise(resolve => setTimeout(resolve,3000))
    return{
      format,
      emails : ref([]),
      openedEmail : ref(null),
      emailSelection: useEmailSelection(),
      selectedScreen: ref('inbox')
    }
  },
  async created(){
    try{
      let res = await axios.get("http://localhost:3000/emails");
      this.emails = res.data;
    } catch(e){
      console.log(e);
    }

  },
  computed: {
    sortedEmails(){
      return this.emails.sort((a,b)=>{
        return a.sentAt < b.sentAt ? 1 : -1;
      })
      
    },
    unarcheivedEmails(){
      if(this.selectedScreen == 'inbox'){
        return this.sortedEmails.filter(e => !e.archived)
      }else{
        return this.sortedEmails.filter(e => e.archived)
      }
      
    }
  },
  methods: {
    selectScreen(newScreen){
      this.selectedScreen = newScreen;
      this.emailSelection.clear();
    },
    openEmail(email){
      this.openedEmail = email
      if(email){
        email.read = true;
        this.updateEmail(email)
      }
    },
    archiveEmail(email){
      email.archived = true;
      this.updateEmail(email)
    },
    changeEmail ({toggleRead,toggleArchive,save,closeModal,changeIndex}){
      let email = this.openedEmail
      if(toggleRead){email.read = !email.read}
      if(toggleArchive){email.archived = !email.archived}
      if(save){this.updateEmail(email)}
      if(closeModal){this.openEmail(email)}

      if(changeIndex){
        let emails = this.unarcheivedEmails
        let currentIndex = emails.indexOf(this.openedEmail)
        let newEmail=emails[currentIndex + changeIndex]
        this.openEmail(newEmail)
      }

    },
    updateEmail(email){
      axios.put(`http://localhost:3000/emails/${email.id}`,email)
    }
  }

};
</script>

<style scoped>

</style>