<template>
  <div class="bulk-action-bar">
    <span class="checkbox">
      <input type="checkbox" 
             :checked="allAreSelected"
             :class="[partialSelection ? 'partial-check' : '']"
             @click="bulkSelect">
    </span>
    <span class="buttons">
      <button @click="emailSelection.markRead()"
              :disabled="[...emailSelection.emails].every(e => e.read)">Mark Read</button>
      <button @click="emailSelection.markUnread()"
              :disabled="[...emailSelection.emails].every(e => !e.read)">Mark Unread</button>
      <button @click="emailSelection.archive()"
              :disabled="numberSelected === 0">Archive</button>

    </span>

    
  </div>
</template>

<script>
  import { useEmailSelection } from '../composition/useEmailSelection';
  import { computed } from 'vue';

  export default {
    setup(props){
      let emailSelection = useEmailSelection();

      let numberSelected = computed(() => {
        return emailSelection.emails.size;
      }) 
      let allAreSelected = computed(() => {
        return props.emails.length == numberSelected.value && numberSelected.value !== 0;
      })
      let partialSelection = computed(() => {
        return numberSelected.value > 0 && !allAreSelected.value;
      })

      let bulkSelect = ()=>{
        if(allAreSelected.value){
          emailSelection.clear();
        }else{
          emailSelection.addMultiple(props.emails)
        }
      }
      

      return { 
        partialSelection, 
        allAreSelected,
        bulkSelect,
        emailSelection,
        numberSelected
      }
    },
    props: {
      emails: {
        type: Array,
        required: true
      }, 
    }
  }
</script>

<style scoped>

</style>