<template>
  <div class="home">
    <div>{{msg}}</div>
    <v-btn
      class="ma-6"
      @click="doMethod()"
      color="pink"
    >
      I am a vuetify button
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
    async doMethod() {
      var openai = new OpenAIApi(new Configuration({ apiKey: "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" }));
      try {
        const response = await openai.createCompletion({
          model: "text-davinci-003",
          prompt: "How to recycle a bottle, give your answer in a single paragraph",
          max_tokens: 100,
          n: 1,
          temperature: 0.8
        });
        console.log(`request cost: ${response.data.usage.total_tokens} tokens`);
        this.msg = response.data.choices[0].text;
        console.log(response);
      } catch (error) {
        console.error("API Error:", error.response);
      }
    }

   
  }
    
}
</script>

<style scoped>
  .home{
    font-weight: bold;
  }
</style>