<template>
  <div class="home">
    <img src="../assets/logo.png">
    <Pet />
  </div>
</template>

<script>
// @ is an alias to /src
import Pet from '@/components/Pet.vue'

export default {
  name: 'home',
  components: {
    Pet
  }
}
</script>
