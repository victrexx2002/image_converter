<template>
    <div>
      <div>
        <h1>Image Converter</h1>
        <p>Convert images online, for free.</p>
        <input type="file" @change="onFileChange" multiple ref="fileInput" />
        <button @click="triggerFileInput">Add More Files</button>
        <div v-for="(fileObj, index) in files" :key="index">
          <div>
            <p>{{ fileObj.file.name }}</p>
            <p>{{ (fileObj.file.size / 1024).toFixed(2) }} KB</p>
            <div>
              <label for="format">Output:</label>
              <select v-model="fileObj.format" id="format">
                <option value="png">PNG</option>
                <option value="jpeg">JPG</option>
                <option value="webp">WEBP</option>
              </select>
            </div>
          </div>
          <div>
            <button @click="convertImage(fileObj, index)">Convert</button>
            <div v-if="fileObj.convertedUrl">
              <a :href="fileObj.convertedUrl" :download="fileObj.outputFileName"><button>Download</button></a>
            </div>
          </div>
        </div>
      </div>
    </div>
</template>
  
<script setup>
  import { FFmpeg } from '@ffmpeg/ffmpeg'
  import { fetchFile, toBlobURL } from '@ffmpeg/util'
  import { ref } from 'vue'
  
  const baseURL = 'https://unpkg.com/@ffmpeg/core-mt@0.12.6/dist/esm'
  
  const files = ref([])

  const ffmpeg = new FFmpeg()
  const fileInput = ref(null)
  
  const triggerFileInput = () => {
    fileInput.value.click()
  }
  
  const onFileChange = (e) => {
    for (let i = 0; i < e.target.files.length; i++) {
      files.value.push({
        file: e.target.files[i],
        format: 'png',
        convertedUrl: null,
        outputFileName: ''
      })
    }
  }
  
  const convertImage = async (fileObj, index) => {
    const file = fileObj.file
  
    if (!file) {
      alert('Please select a file first')
      return
    }

    ffmpeg.on('log', ({ message: msg }) => {
      message.value = msg
    })
  
    await ffmpeg.load({
      coreURL: await toBlobURL(`${baseURL}/ffmpeg-core.js`, 'text/javascript'),
      wasmURL: await toBlobURL(`${baseURL}/ffmpeg-core.wasm`, 'application/wasm'),
      workerURL: await toBlobURL(`${baseURL}/ffmpeg-core.worker.js`, 'text/javascript')
    })
  
    const fileName = file.name
    const fileExt = fileName.split('.').pop()
    const inputFileName = `input.${fileExt}`
    const outputFileName = fileName.replace(`.${fileExt}`, `.${fileObj.format}`)
    fileObj.outputFileName = outputFileName

    await ffmpeg.writeFile(inputFileName, await fetchFile(file))
    await ffmpeg.exec(['-i', inputFileName, outputFileName])
  
    const data = await ffmpeg.readFile(outputFileName)
    fileObj.convertedUrl = URL.createObjectURL(new Blob([data.buffer], { type: `image/${fileObj.format}` }))
  }
</script>
  
<style scoped>

</style>