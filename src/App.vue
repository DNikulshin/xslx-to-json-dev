<template>
  <div class="app">
    <div class="text-center">
      <img src="./assets/fa-bg.jpg" alt="fa-bg" class="img">
    </div>
    <div class="container">
      <div class="mb-3">
        <label for="formFile" class="form-label">Выберите файл .xslx для загрузки</label>
        <input
            @change="addFile($event)"
            placeholder="Upload file"
            accept=".csv,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel"
            class="form-control w-50"
            type="file"
            id="formFile"
        >
      </div>
      <div v-if="!loading" class="content">
        <input type="search" v-model="findText" placeholder="Введите текст поиска.." v-show="fileList.length > 0"
               class="search" ref="search">
        <strong>Всего:&nbsp;{{ filteredFileList.length }}</strong>

        <TransitionGroup class="list-group" name="list" tag="ul">
          <li v-for="(item,idx) in filteredFileList" :key="item"
              class="list-group-item my-1 border box-shadow item-hover">
            <small class="">#{{ idx + 1 }}&nbsp;</small>
            <span class="mx-3">{{ Object.values(item).join(' ') }}</span>
          </li>
        </TransitionGroup>
        <div style="text-align: center; color: #FF3D00;" v-show="filteredFileList.length === 0">
          Нет результатов поиска...
          <hr style="border: 1px solid  #FF3D00; width: 50%;"/>
        </div>
      </div>
      <div v-else style="text-align: center;"><span class="loader"></span></div>


    </div>

  </div>

</template>

<script setup>
import {ref, onMounted, computed, nextTick} from 'vue'
import useDebounce from './hooks/UseDebounce.js'
import * as XLSX from 'xlsx'

const arrayBuffer = ref()
const fileList = ref([])
const file = ref(null)
const loading = ref(false)
const findText = useDebounce('', 300)
const search = ref(null)

onMounted(() => {
  nextTick(() => {
    search.value?.focus()
  })
  if (localStorage.getItem('resultArray')) {
    fileList.value = JSON.parse(localStorage.getItem('resultArray'))
  }

})
const addFile = (e) => {
  loading.value = true
  fileList.value = []
  localStorage.removeItem('resultArray')
  file.value = e.target.files[0]
  const fileReader = new FileReader()
  fileReader.readAsArrayBuffer(file.value)

  fileReader.onload = () => {
    arrayBuffer.value = fileReader.result
    const data = new Uint8Array(arrayBuffer.value)
    const arr = []
    for (let i = 0; i !== data.length; ++i)
      arr[i] = String.fromCharCode(data[i])
    const bStr = arr.join('')
    const workbook = XLSX.read(bStr, {type: 'binary'})
    const first_sheet_name = workbook.SheetNames[0]
    const worksheet = workbook.Sheets[first_sheet_name]
    fileList.value = XLSX.utils.sheet_to_json(worksheet, {raw: true})
    localStorage.setItem('resultArray', JSON.stringify(fileList.value))
    loading.value = false
  }
}

const filteredFileList = computed(() => {
  if (!findText.value) {
    return fileList.value
  }
  return fileList.value.filter((item) => {
  
    if (
        Object.values(item).join(' ').toLowerCase().includes(findText.value.toLowerCase())
    ) {
      return Object.values(item).join(' ')
    }
  })
})

</script>

<style>
body {
  position: relative;
  background: rgba(0, 0, 0, 0.1);
}


.img {
  position: fixed;
  top: 1rem;
  right: 2rem;
  width: 100px;
  height: 100px;
}

.app {
  padding: .5rem;
  margin: 0 auto;
  width: 100%;
  height: 100%;
}

.add-file {
  margin: .5rem;
}

.content {
  position: relative;
  width: 100%;
  height: 100%;
}

.search {
  width: 50%;
  padding: .5rem;
  margin: 1rem;
}

.loader {
  top: 50% !important;
  position: relative;
  width: 48px;
  height: 48px;
}

.loader:before,
.loader:after {
  content: "";
  display: block;
  border: 32px solid transparent;
  border-top-color: #676363;
  position: absolute;
  top: 50% !important;
  left: 50% !important;
  animation: weld-rotate 2s infinite ease-in;
}

.loader:before {
  border-color: transparent transparent transparent #FF3D00;
  animation-delay: 0.5s;
}

@keyframes weld-rotate {
  0%, 25% {
    transform: rotate(0deg)
  }
  50%, 75% {
    transform: rotate(180deg)
  }
  100% {
    transform: rotate(360deg)
  }
}

.list-enter-active,
.list-leave-active {
  opacity: 1;
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.box-shadow {
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.5);
}

.item-hover:hover {
  background: rgba(10, 225, 10, 0.2) !important;
}

</style>
