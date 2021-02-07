<template>
  <section class="text-gray-600 body-font">
    <div
      class="container mx-auto flex px-10 py-10 md:flex-row flex-col items-center"
    >
      <div
        class="lg:flex-grow md:w-1/2 lg:pr-24 md:pr-16 flex flex-col md:items-start md:text-left mb-16 md:mb-0 items-center text-center"
      >
        <h1
          class="title-font sm:text-4xl text-3xl mb-4 font-medium text-gray-900"
        >
          Video to gif Converter
        </h1>
        <p class="mb-8 leading-relaxed">
          choose any mp4 video to convert
        </p>
        <div class="flex  flex-col justify-center">
          <input type="file" accept="video/*" @change="loadVid" class="p-5" />
          <div class="p-5">
            <button
              v-if="video"
              @click="convertToGif"
              class="inline text-white bg-indigo-500 border-0 py-2 px-6 focus:outline-none hover:bg-indigo-600 rounded text-lg"
            >
              Convert
            </button>
          </div>
          <div class="p-5">
            <a
              v-if="gif"
              :href="gif"
              target="_blank"
              download="g.gif"
              class="inline text-white bg-indigo-500 border-0 py-2 px-6 focus:outline-none hover:bg-indigo-600 rounded text-lg"
              >Download GIF</a
            >
          </div>
        </div>
      </div>
      <div class="lg:max-w-lg lg:w-full md:w-1/2 w-5/6">
        <video class="p-2" width="400" controls v-if="video">
          <source :src="video" type="video/mp4" />
        </video>
        <img class="p-2" v-if="gif" :src="gif" alt="gif" width="400" />
      </div>
    </div>
  </section>
</template>

<script>
import "./index.css";

const { createFFmpeg, fetchFile } = require("@ffmpeg/ffmpeg");

const ffmpeg = createFFmpeg({ log: true });

export default {
  name: "App",
  data() {
    return {
      video: null,
      ready: false,
      gif: null,
    };
  },
  created() {
    this.load();
  },
  methods: {
    async convertToGif() {
      if (!this.ready) {
        console.log("not ready");
        return;
      }
      ffmpeg.FS("writeFile", "test.mp4", await fetchFile(this.video));
      await ffmpeg.run(
        "-i",
        "test.mp4",
        "-t",
        "5",
        "-ss",
        "5",
        "-f",
        "gif",
        "out.gif"
      );
      const filedata = ffmpeg.FS("readFile", "out.gif");
      const url = URL.createObjectURL(new Blob([filedata.buffer]));
      this.gif = url;
    },
    loadVid(event) {
      this.video = URL.createObjectURL(event.target.files[0]);
    },
    async load() {
      await ffmpeg.load();
      this.ready = true;
      console.log("ready");
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
