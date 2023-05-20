<template>
  <div class="home">
    <div>{{msg}}</div>
    <v-btn
      class="ma-6"
      @click="doMethod()"
      color="green"
    >
      I am a vuetify button
    </v-btn>
    <component1 class="mt-5"></component1>
  </div>
</template>

<script>
import component1 from '@/components/component1.vue'

export default {
  components: {
    component1
  },
  data() {
    return {
      msg: "This is the home page"
    }
  },
  //before the component is mounted
  created(){
    console.log("Created")
  },
  computed: {
  },
  methods: {
    doMethod(){
      console.log("button clicked")
    }
  }
}
</script>

<style scoped>
  .home{
    font-weight: bold;
  }
</style>