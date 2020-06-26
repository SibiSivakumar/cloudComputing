<template>
  <div class="hello">
    <h1>CC Video Uploader</h1>

    <div v-if="!image">
      <h2>Select an Video</h2>
      <input type="file" @change="onFileChange">
    </div>
    <div v-else>
      <img :src="image" />
      <button v-if="!uploadURL" @click="removeImage">Remove Video</button>
      <button v-if="!uploadURL" @click="uploadImage">Upload Video</button>
    </div>
    <h2 v-if="uploadURL">Success! Video uploaded to:</h2>
    <a :href="uploadURL">{{ uploadURL }}</a>
    <div>
    <a href="https://v2-alpha.d1cxb96fgndu6o.amplifyapp.com/">Visit S3 Explorer!</a>
    </div>
  </div>
</template>

<script>

import axios from 'axios'
const MAX_IMAGE_SIZE = 1000000000

export default {
  name: 's3uploader',
  data () {
    return {
      image: '',
      uploadURL: ''
    }
  },
  methods: {
    onFileChange (e) {
      let files = e.target.files || e.dataTransfer.files
      if (!files.length) return
      this.createImage(files[0])
    },
    createImage (file) {
      // var image = new Image()
      let reader = new FileReader()

      reader.onload = (e) => {
        console.log('length: ', e.target.result.includes('data:video/mp4'))
        if (!e.target.result.includes('data:video/mp4')) {
          if (!e.target.result.includes('data:video/avi')){
            return alert('Wrong file type - MP4 & AVI only.')
          }				
        }
        if (e.target.result.length > MAX_IMAGE_SIZE) {
          return alert('Video is too large - 100Mb maximum')
        }

        this.image = e.target.result
      }
      reader.readAsDataURL(file)
    },
    removeImage: function (e) {
      console.log('Remove clicked')
      this.image = ''
    },
    uploadImage: async function (e) {
      console.log('Upload clicked')

      // Get the presigned URL
      const response = await axios({
        method: 'GET',
        url: `https://clw1r2km5b.execute-api.ap-south-1.amazonaws.com/Prod/`
      })

      console.log('Response: ', response.data)
      console.log('Uploading: ', this.image)

      let binary = atob(this.image.split(',')[1])
      let array = []
      for (var i = 0; i < binary.length; i++) {
        array.push(binary.charCodeAt(i))
      }
      let blobData = new Blob([new Uint8Array(array)], {type: 'video/mp4'})
      console.log('Uploading to: ', response.data.uploadURL)

      const result = await fetch(response.data.uploadURL, {
        method: 'PUT',
        body: blobData
      })

      console.log('Result: ', result)

      // Final URL for the user doesn't need the query string params
      this.uploadURL = response.data.uploadURL.split('?')[0]
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
#app {
  text-align: center;
}
img {
  width: 30%;
  margin: auto;
  display: block;
  margin-bottom: 10px;
}
button {
}
</style>
