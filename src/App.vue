<template>
  <div id="app">
    <h1>File Uploader</h1>
    <input @change="selectFile" type="file" />
    <div class="images">
      <ul v-if="images.length > 0">
        <li v-for="(image, index) in images" :key="index">
          <a :href="image.url" target="_blank" rel="noopener">
            {{ image.name }}
          </a>
        </li>
      </ul>
      <template v-else>
        <span>empty </span>
      </template>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue"
import { storage } from "@/firebase"

export default Vue.extend({
  name: "App",
  async mounted() {
    await this.fetchImages()
  },
  data: () => ({
    images: [] as any,
  }),
  computed: {
    imagesRef() {
      return storage.ref().child("images/")
    },
  },
  methods: {
    async fetchImages() {
      const imageList = await storage
        .ref()
        .child("images")
        .listAll()

      const imagePromises = imageList.items.map(async (image) => {
        const url = await image.getDownloadURL()
        return {
          name: image.name,
          url,
        }
      })
      this.images = await Promise.all(imagePromises)
    },
    selectFile(e: Event) {
      const files = (e.target as HTMLInputElement).files
      if (!files) {
        return
      }

      const file = files[0]
      this.uploadFile(file)
    },
    uploadFile(file: File) {
      const refName = new Date()
      const uploadfileRef = this.imagesRef.child(file.name)
      uploadfileRef.put(file).then((snapshot: any) => {
        console.log("uploaded")
        this.fetchImages()
      })
    },
  },
})
</script>

<style lang="scss"></style>
