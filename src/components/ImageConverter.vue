<template>
  <div class="main">
    <div class="container">
      <h1>Free Image Converter</h1>
      <p>
        Convert your images files into JPG, PNG, SVG, or PDF like magic. Use
        Canva's free online image converter to your photos into a format suited
        to your platform or project, without worrying about losing image
        quality.
      </p>
      <div class="add-container">
        <div>
          <!-- <input type="file" @change="onFileChange" multiple  /> -->
        </div>
        <div>
          <button @click="triggerFileInput" class="btn--btn" ref="fileInput">
            Upload Files
          </button>
        </div>
      </div>

      <div
        v-for="(fileObj, index) in files"
        :key="index"
        class="file-container"
      >
        <div>
          <!-- name of file -->
          <div class="file-name">
            <div>
              <p>{{ fileObj.file.name }}</p>
              <p>{{ (fileObj.file.size / 1024).toFixed(2) }} KB</p>
            </div>
          </div>

          <div class="file-wrapper">
            <!-- output format -->
            <div>
              <div>
                <label for="format">Output:</label>
                <select v-model="fileObj.format" id="format">
                  <option value="png">PNG</option>
                  <option value="jpeg">JPG</option>
                  <option value="webp">WEBP</option>
                </select>
              </div>
            </div>
            <!-- download and convert buttons -->
            <div class="btns">
              <button @click="convertImage(fileObj, index)" class="btn--btn">
                Convert
              </button>
              <div v-if="fileObj.convertedUrl">
                <a
                  :href="fileObj.convertedUrl"
                  :download="fileObj.outputFileName"
                  ><button class="btn--btn">Download</button></a
                >
              </div>
            </div>
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
    });
  }
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
