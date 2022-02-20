<template>
  <div class="modal">
    <div class="overlay" @click="emit('closeModal')"></div>
    <div class="modal-card">
        <slot />
    </div>
  </div>
</template>

<script>
import { onBeforeUnmount } from '@vue/runtime-core'
import useKeydown from "../composition/useKeydown"
  export default {
    setup(props, {emit}){
      useKeydown([
        {key: 'Escape', fn: ()=>{emit('closeModal')}},
        {key: 'Enter', fn: ()=>{console.log('a different function')}},
      ])

      return {
        emit,
      }
    }
  }
</script>

<style scoped>

</style>