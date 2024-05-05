<script setup>
import { ref, watch, computed, reactive } from 'vue'
import { Codemirror } from 'vue-codemirror'
import { oneDark } from '@codemirror/theme-one-dark'
import { useBrowserLocation } from '@vueuse/core'
import debounce from './helpers/debounce'
import moment from 'moment'
import LangList from './components/LangList.vue'
import axios from 'axios'
axios.defaults.baseURL = "http://127.0.0.1:5000/"

const location = useBrowserLocation()

const state = reactive(
  {
    code: '',
    lang: '',
    timestamp: null
  }
)

let result = ref("the results will be displayed here")

const commit = () => {
  axios.post('/code_api', state.code).then(res =>{
    console.log(res);
    result.value = res.data;
  })
  console.log(result);
}

const extensions = ref([oneDark])

const updatedAt = computed(() => {
  var date = moment(state.timestamp);
  if ( !date.isValid() ) return ''
  return date.format("Do MMM YYYY, HH:mm:ss")
})

const onChange = debounce((val) => {
  state.timestamp = new Date();
  window.location.hash = btoa(JSON.stringify(state))
}, 500)

const restoreState = () => {
  var restored_state = {}

  try {
    restored_state = JSON.parse(atob(location.value.hash.substring(1)));
  } catch (e) {}

  Object.keys(state).forEach((key) => {
    state[key] = restored_state[key] || state[key]
  })
}

const setExtensions = (language) => {
  extensions.value = [oneDark, language]
}

watch(location, () => {
  restoreState()
})

restoreState()

</script>

<template>

<header>
  <div class="container h-100 d-flex align-items-center">
    <LangList v-model="state.lang" @onLoadLanguage="setExtensions" />
    <button class="ms-auto" @click="commit">提交</button>
  </div>
</header>

<div class="flex-grow-1 position-relative overflow-auto">
  <codemirror
    v-model="state.code"
    :style="{ height: '100%' , fontSize: '18px'}"
    :autofocus="true"
    :indent-with-tab="true"
    :tab-size="2"
    :extensions="extensions"
    @change="onChange"
  />
</div>

<div class="result">
  <pre style="font-size: Consolas;">{{ result }}</pre>
</div>
  
</template>

<style>
.result {
  left: 0;
  bottom: 0;
  width: 100%;
  max-height: 200px;
  min-height: 200px;
  border-top: 1px;
  border-left: 0;
  border-bottom: 0;
  border-right: 0;
  border-top-color: white;
  border-style:solid;
  background-color: #1e2227;
  font-family: Consolas;
  font-size: 18px;
  color: #abb2bf;
  padding: 5px 0 0 30px;
  overflow: auto;
}
.cm-content {
  font-family: Consolas;
}
</style>
