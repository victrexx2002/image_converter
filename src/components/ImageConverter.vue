<template>
  <div class="homepage">
    <div class="container">
      <h1>Image Converter</h1>
      <p class="container-txt">
        Convert your images files into JPG, PNG, SVG, or PDF like magic. Use
        Canva's free online image converter to your photos into a format suited
        to your platform or project, without worrying about losing image
        quality.
      </p>
      <input type="file" @change="onFileChange" multiple ref="fileInput" />
      <button @click="triggerFileInput">
        <span class="text">Add More Files</span>
      </button>

      <div v-for="(fileObj, index) in files" :key="index" class="convert_container">
        <div>
          <div class="dash_container">
            <p>{{ fileObj.file.name }}</p>
            <p>{{ (fileObj.file.size / 1024).toFixed(2) }} KB</p>
          </div>
          <div class="label_container">
            <div class="custom-select">
              <label for="format">Output:</label>
              <div class="select-selected" @click="toggleDropdown(index)">
                {{ fileObj.format.toUpperCase() }}
              </div>
              <div v-show="fileObj.isOpen" class="select-items">
                <div 
                  v-for="option in options" 
                  :key="option" 
                  @click="selectOption(option, index)"
                  :class="{ 'same-as-selected': fileObj.format === option }">
                  {{ option.toUpperCase() }}
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="btns">
          <button @click="convertImage(fileObj, index)" class="convert_btn">Convert</button>
          <div v-if="fileObj.convertedUrl">
            <a :href="fileObj.convertedUrl" :download="fileObj.outputFileName">
              <button class="download_btn">Download</button>
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>




<script setup>
import { FFmpeg } from "@ffmpeg/ffmpeg";
import { fetchFile, toBlobURL } from "@ffmpeg/util";
import { ref } from "vue";

const baseURL = "https://unpkg.com/@ffmpeg/core-mt@0.12.6/dist/esm";

const files = ref([]);
const options = ['png', 'jpeg', 'webp'];

const ffmpeg = new FFmpeg();
const fileInput = ref(null);

const triggerFileInput = () => {
  fileInput.value.click();
};

const onFileChange = (e) => {
  for (let i = 0; i < e.target.files.length; i++) {
    files.value.push({
      file: e.target.files[i],
      format: "png",
      convertedUrl: null,
      outputFileName: "",
      isOpen: false
    });
  }
};

const toggleDropdown = (index) => {
  files.value[index].isOpen = !files.value[index].isOpen;
};

const selectOption = (option, index) => {
  files.value[index].format = option;
  files.value[index].isOpen = false;
};

const convertImage = async (fileObj, index) => {
  const file = fileObj.file;

  if (!file) {
    alert("Please select a file first");
    return;
  }

  ffmpeg.on("log", ({ message: msg }) => {
    message.value = msg;
  });

  await ffmpeg.load({
    coreURL: await toBlobURL(`${baseURL}/ffmpeg-core.js`, "text/javascript"),
    wasmURL: await toBlobURL(`${baseURL}/ffmpeg-core.wasm`, "application/wasm"),
    workerURL: await toBlobURL(
      `${baseURL}/ffmpeg-core.worker.js`,
      "text/javascript"
    ),
  });

  const fileName = file.name;
  const fileExt = fileName.split(".").pop();
  const inputFileName = `input.${fileExt}`;
  const outputFileName = fileName.replace(`.${fileExt}`, `.${fileObj.format}`);
  fileObj.outputFileName = outputFileName;

  await ffmpeg.writeFile(inputFileName, await fetchFile(file));
  await ffmpeg.exec(["-i", inputFileName, outputFileName]);

  const data = await ffmpeg.readFile(outputFileName);
  fileObj.convertedUrl = URL.createObjectURL(
    new Blob([data.buffer], { type: `image/${fileObj.format}` })
  );
};
</script>



<style scoped></style>
