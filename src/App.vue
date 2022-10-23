<script setup>
import {reactive, ref, onMounted} from 'vue';
import { useDropzone } from "vue3-dropzone";
import axios from 'axios';

const options = reactive({
  multiple: false,
  onDrop,
  accept: '.csv'
});

const fileIsLoaded = ref(false);
const statusS3 = ref('failed');
const urlS3 = ref('none');
const loading = ref(false);
const fileSelected = ref(null);
const ArchivoProcesado = ref(false);

const {
  getRootProps,
  getInputProps,
  isDragActive,
  isDragReject,
  open
} = useDropzone(options);


async function onDrop(acceptFiles, rejectReasons) {
  console.log(acceptFiles, statusS3.value);
  if(acceptFiles){
    if( statusS3.value === 'success' ){
      fileIsLoaded.value = true;
      fileSelected.value = acceptFiles[0];
      console.log(fileSelected.value);

    }
  }
  console.log(rejectReasons);
}


async function processFile(){
  console.log('subiendo archivo');

  await axios({
    method: 'put',
    url: urlS3.value,
    data:fileSelected.value[0],
    headers: {
      'Content-Type': 'text/csv'
    }
  });
  fileIsLoaded.value = false;
  loading.value = true;

  setTimeout(()=>{
    ArchivoProcesado.value = true;
  },5000);

}

async function reset(){
  window.location.href = '/';
}


let formData = new FormData();

onMounted(async() => {
  // Create the S3 Signed URL.
  const preSignedS3URLResponse = await axios.post("https://gt47vf0wmj.execute-api.us-east-1.amazonaws.com/production/generate-signed-url",{
    "fileName":`carga_${Date.now().toString()}.csv`
  })
  const {
    status,
    url
  } = preSignedS3URLResponse.data;

  urlS3.value = url;
  statusS3.value = status;
});
</script>

<template>
  <div class="main">
    <div>
      <h2>Pyme Data Gathering Tool</h2>
      <h4>Carga de datos </h4>
    </div>
    <div>
      <img src="/src/assets/logo-min_1.png" class="logo-min">
    </div>
  </div>
    <div class="wrapper">
      <div v-if="!ArchivoProcesado" class="file-container">
        <div v-if="fileIsLoaded">
          <h3>Archivo seleccionado:<br> {{ fileSelected.name }}</h3>
        </div>
        <div v-if="!loading && !fileIsLoaded">
          <div v-bind="getRootProps()">
            <input v-bind="getInputProps()" />
            <img src="src/assets/upload.png" class="upload-image">
            <p v-if="isDragActive" class="text-center">Arrastre su archivo aquí ...</p>
            <h3 class="title text-center">Seleccione el archivo</h3>
            <p class="small text-center">o arrastre y suelte aquí</p>
          </div>
        </div>
        <div v-else-if="loading">
          <img src="src/assets/load.gif" class="loading-image">
          <h3 class="title text-center">Procesando archivo...</h3>
        </div>
      </div>
      <div v-else-if="ArchivoProcesado">
        <h3><a href="#">descargar archivo</a></h3>
      </div>
      <div v-if="ArchivoProcesado">
          <button class="button-send"   @click="reset" ><i class="fa fa-plus"></i>Subir un nuevo archivo</button>
      </div>
      <div v-if="!ArchivoProcesado">
        <button class="button-send" :disabled="!fileIsLoaded"  @click="processFile" ><i class="fa fa-gear"></i> Procesar</button>
      </div>
    </div>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.button-send:disabled,
.button-send[disabled]{
  background-color: #cccccc;
  color: #fff;
  cursor: default;
}

.title{
  font-size: 30px;
}

h4{
  font-size: 20px;
}
h2{
  font-size: 28px;
}

.main{
  margin-top: 30px;
  margin-bottom: 100px;
  display: flex;
  justify-content: space-between;
  padding-bottom: 20px;
  border-bottom: 1px solid #DEDEDE;
}

.upload-image{
  max-width: 100px;
}

.loading-image{
  max-width: 70px;
}


.logo-min{
  max-width: 80px;
}

.button-send{
  border-radius: 10px;
  border:none;
  margin-top: 30px;
  padding: 10px 20px;
  font-size:18px;
  cursor: pointer;
  color: #fff;
  background-color: #3771B3;
}

.text-center{
  text-align: center;
}
.small{
  margin-top: 10px;
  color: #999;
}
.logo {
  display: block;
  margin: 0 auto 2rem;
}

.file-container{
  width:600px;
  height: 300px;
  border: 3px dashed #000;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
}


@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
  }

  .logo {
    margin: 0 2rem 0 0;
  }

   .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
    align-items: center;
    flex-direction: column;
     margin-bottom: 20px;
  }
}
</style>
