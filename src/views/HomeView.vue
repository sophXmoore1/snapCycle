<template>
  <div class="home">
    <div>{{msg}}</div>
    <v-btn
      class="ma-6"
      href="/camera"
      color="green"
    >
      Go To Camera
    </v-btn>
    <component1 class="mt-5"></component1>
  </div>
</template>

<script>
import component1 from '@/components/component1.vue'
import { Configuration, OpenAIApi } from 'openai';

export default {
  components: {
    component1
  },
  data() {
    return {
      msg: "Welcome to snapCycle"
    }
  },
  //before the component is mounted
  created(){
    console.log("Created")
  },
  computed: {
  },
  methods: {
  }
    
}
</script>

<style scoped>
  .home{
    font-weight: bold;
  }
</style>