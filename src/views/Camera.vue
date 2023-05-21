<template>
  <div id="app" class="web-camera-container">
    <div v-if="item">
      item: {{item}}
    </div>
    <div v-if="recycleClassification">
      {{recycleClassification}}
    </div>
    <div v-show="isCameraOpen && isLoading" class="camera-loading">
      <ul class="loader-circle">
        <li></li>
        <li></li>
        <li></li>
      </ul>
    </div>
    
    <div v-if="isCameraOpen" v-show="!isLoading" class="camera-box" :class="{ 'flash' : isShotPhoto }">
      
      <div class="camera-shutter" :class="{'flash' : isShotPhoto}"></div>
        
      <video v-show="!isPhotoTaken" ref="camera" :width="450" :height="337.5" autoplay></video>
      
      <canvas v-show="isPhotoTaken" id="photoTaken" ref="canvas" :width="450" :height="337.5"></canvas>
    </div>
    
    <div v-if="isCameraOpen && !isLoading" class="camera-shoot">
      <button type="button" class="button" @click="takePhoto">
        <img src="https://img.icons8.com/material-outlined/50/000000/camera--v2.png">
      </button>
    </div>
    
    <div v-if="isPhotoTaken && isCameraOpen" class="camera-download">
      <a id="downloadPhoto" download="my-photo" class="button" role="button" @click="downloadImage">
        Analyze
      </a>
    </div>
  </div>
</template>

<script>
import component1 from '@/components/component1.vue'
import axios from 'axios';
import { Configuration, OpenAIApi } from 'openai';

export default {
  components: {
    component1
  },
  data() {
    return {
      isCameraOpen: false,
      isPhotoTaken: false,
      isShotPhoto: false,
      isLoading: false,
      link: '#',
      item: null,
      recycleClassification: null
    }
  },
  //before the component is mounted
  created(){
    this.createCameraElement()
    this.isCameraOpen = true
  },
  computed: {
  },
  methods: {
    createCameraElement() {
      this.isLoading = true;
      
      const constraints = (window.constraints = {
				audio: false,
				video: true
			});


    navigator.mediaDevices
      .getUserMedia(constraints)
      .then(stream => {
        this.isLoading = false;
        this.$refs.camera.srcObject = stream;
      })
      .catch(error => {
        this.isLoading = false;
        alert("May the browser didn't support or there is some errors.");
      });
    },
    
    stopCameraStream() {
      let tracks = this.$refs.camera.srcObject.getTracks();

			tracks.forEach(track => {
				track.stop();
			});
    },
    
    takePhoto() {
      if(!this.isPhotoTaken) {
        this.isShotPhoto = true;

        const FLASH_TIMEOUT = 50;

        setTimeout(() => {
          this.isShotPhoto = false;
        }, FLASH_TIMEOUT);
      }
      
      this.isPhotoTaken = !this.isPhotoTaken;
      
      const context = this.$refs.canvas.getContext('2d');
      context.drawImage(this.$refs.camera, 0, 0, 450, 337.5);
    },
    
    async downloadImage() {
      const download = document.getElementById("downloadPhoto");
      const canvas = document.getElementById("photoTaken").toDataURL()
      download.setAttribute("href", canvas);
      var item = await this.useFlask()
      var prompt = "How do I recycle or compost, if applicable, the following item: " + this.item + ", give your answer in a single paragraph"
      console.log("prompt", prompt)
      await this.useOpenAI(prompt)
    },
    async useFlask(){
      const path = 'http://localhost:5001/ping';
      await axios.get(path)
        .then((res) => {
          let result = res.data
          this.item = result.split(": ")[1]
          console.log(this.item)
          return result.split(": ")[1]
        })
        .catch((error) => {

          console.error(error);
        });
    },

    async useOpenAI(prompt) {
      var openai = new OpenAIApi(new Configuration({ apiKey: "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" }));
      try {
        const response = await openai.createCompletion({
          model: "text-davinci-003",
          prompt: prompt,
          max_tokens: 100,
          n: 1,
          temperature: 0.8
        });
        console.log(`request cost: ${response.data.usage.total_tokens} tokens`);
        this.recycleClassification = response.data.choices[0].text;
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