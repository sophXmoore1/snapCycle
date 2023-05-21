<template>
  <v-btn @click="useFlask()">
    use Flask
  </v-btn>
  <div id="app" class="web-camera-container">
    <div class="camera-button">
        <button type="button" class="button is-rounded" :class="{ 'is-primary' : !isCameraOpen, 'is-danger' : isCameraOpen}" @click="toggleCamera">
          <span v-if="!isCameraOpen">Open Camera</span>
          <span v-else>Close Camera</span>
      </button>
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
      <a id="downloadPhoto" :download="imageId+'.jpg'" class="button" role="button" @click="downloadImage">
        Download
      </a>
    </div>
  </div>
</template>

<script>
import component1 from '@/components/component1.vue'
import Replicate from "replicate";
import axios from 'axios';

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
      imageId: null
    }
  },
  //before the component is mounted
  created(){
    console.log("Created")
  },
  computed: {
  },
  methods: {
    toggleCamera() {
      if(this.isCameraOpen) {
        this.isCameraOpen = false;
        this.isPhotoTaken = false;
        this.isShotPhoto = false;
        this.stopCameraStream();
      } else {
        this.isCameraOpen = true;
        this.createCameraElement();
      }
    },
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
    
    downloadImage() {
      const download = document.getElementById("downloadPhoto");
      const canvas = document.getElementById("photoTaken").toDataURL()
      // const blob = this.b64toBlob(canvas, contentType);
      // const blobUrl = URL.createObjectURL(blob);
      this.useFlask()     
      // // atob to base64_decode the data-URI
      // var image_data = atob(canvas.split(',')[1]);
      // console.log("image_data", image_data)
      // // Use typed arrays to convert the binary data to a Blob
      // var arraybuffer = new ArrayBuffer(image_data.length);
      // var view = new Uint8Array(arraybuffer);
      // for (var i=0; i<image_data.length; i++) {
      //     view[i] = image_data.charCodeAt(i) & 0xff;
      // }
      // try {
      //   // This is the recommended method:
      //   var blob = new Blob([arraybuffer], {type: 'application/octet-stream'});
      // } catch (e) {
      //   // The BlobBuilder API has been deprecated in favour of Blob, but older
      //   // browsers don't know about the Blob constructor
      //   // IE10 also supports BlobBuilder, but since the `Blob` constructor
      //   //  also works, there's no need to add `MSBlobBuilder`.
      //   var bb = new (window.WebKitBlobBuilder || window.MozBlobBuilder);
      //   bb.append(arraybuffer);
      //   var blob = bb.getBlob('application/octet-stream'); // <-- Here's the Blob
      // }
      // var url = (window.webkitURL || window.URL).createObjectURL(blob);
      // console.log("url", url)
      //download.setAttribute("href", canvas);
    },

    b64toBlob(b64Data, contentType='', sliceSize=512){
      const byteCharacters = atob(b64Data);
      const byteArrays = [];

      for (let offset = 0; offset < byteCharacters.length; offset += sliceSize) {
        const slice = byteCharacters.slice(offset, offset + sliceSize);

        const byteNumbers = new Array(slice.length);
        for (let i = 0; i < slice.length; i++) {
          byteNumbers[i] = slice.charCodeAt(i);
        }

        const byteArray = new Uint8Array(byteNumbers);
        byteArrays.push(byteArray);
      }

      const blob = new Blob(byteArrays, {type: contentType});
      return blob;
    },

    useFlask(){
      const path = 'http://localhost:5001/ping';
      axios.get(path)
        .then((res) => {
          console.log(res.data)
        })
        .catch((error) => {

          console.error(error);
        });
    },

    dataURItoBlob(dataURI){
    // convert base64/URLEncoded data component to raw binary data held in a string
    var byteString;

    if(dataURI.split(',')[0].indexOf('base64') >= 0)
        byteString = atob(dataURI.split(',')[1]);
    // else
    //     byteString = unescape(dataURI.split(',')[1]);

    // separate out the mime component
    var mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0];

    // write the bytes of the string to a typed array
    var ia = new Uint8Array(byteString.length);
    for(var i = 0; i < byteString.length; i++)
    {
        ia[i] = byteString.charCodeAt(i);
    }

      return new Blob([ia], {type: mimeString});
    }
  }
}
</script>

<style scoped>
  .home{
    font-weight: bold;
  }
</style>