<script setup>
import {computed, ref} from 'vue';
import axios from 'axios';

let Long_Url = ref('');
let Shorted_Url = ref('');
let DisplayInformation = ref('');
let Error_message = ref('');
let Target_Url = ref('');

axios.defaults.headers.post['Content-Type'] = 'application/json';
axios.defaults.baseURL = 'http://localhost:8080';

function shortenUrl() {
  if (isUrl(Long_Url.value)) {
    Error_message.value = '';
    axios.post(`/api/links?longUrl=${Long_Url.value}`).then(response => {
      console.log(response.data);
      Shorted_Url.value = axios.defaults.baseURL + "/" + response.data.shortUrl;
    }).catch(error => {
      Error_message.value = error.message;
    });
  } else {
    Error_message.value = 'Please input a valid URL.';
  }
}

function displayAll() {
  axios.get(`/api/logs?target=all`).then(response => {
    DisplayInformation.value = JSON.stringify(response.data, null, 2);
  }).catch(error => {
    Error_message.value = error.message;
  });
}

function displayTargetInformation() {
  const shortUrl = isShortedUrl(Target_Url.value);
    axios.get(`/api/logs?target=${shortUrl}`).then(response => {
      DisplayInformation.value = JSON.stringify(response.data, null, 2);
    }).catch(error => {
      Error_message.value = error.message;
    });
}

function isShortedUrl(str) {
  let pattern = new RegExp('^' + axios.defaults.baseURL.replace(/https?:\/\//, '') + '/[a-zA-Z0-9]{6}$');
  if (pattern.test(str)) {
    return str.slice(-6);
  }
  return str;
}

function isUrl(str) {
  let pattern = new RegExp('^(https?:\\/\\/)?'+ // protocol
      '(([a-zA-Z\\d]([a-zA-Z\\d-]*[a-zA-Z\\d])*)\\.)+[a-zA-Z]{2,}' + // domain name and TLD
      '(\\:\\d+)?(\\/[-a-zA-Z\\d%_.~+]*)*'+ // port and path
      '(\\?[;&a-zA-Z\\d%_.~+=-]*)?'+ // query string
      '(\\#[-a-zA-Z\\d_]*)?$','i'); // fragment locator
  return pattern.test(str);
}

const parsedDisplayInformation = computed(() => {
  try {
    return JSON.parse(DisplayInformation.value);
  } catch (e) {
    return []; // 如果无法解析为JSON，则返回空数组
  }
});
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />
  </header>

  <main>
    <p>Please input the URL you need to shorten:</p>
    <input v-model="Long_Url" placeholder="Origin URL">
    <el-button type="primary" @click="shortenUrl">Shorten</el-button>
    <div v-if="Error_message">{{ Error_message }}</div>
    <el-link :href="Shorted_Url" v-if="Shorted_Url">Shortened URL: {{ Shorted_Url }}</el-link>
    <p></p>
    <el-button type="primary" @click="displayAll">Display All Information</el-button>
    <input v-model="Target_Url" placeholder="Target URL">
    <el-button type="primary" @click="displayTargetInformation">Display Target Information</el-button>

    <!-- Display information using Element Plus table -->
    <el-table :data="parsedDisplayInformation" style="width: 100%" v-if="DisplayInformation">
      <el-table-column prop="id" label="ID" width="80"></el-table-column>
      <el-table-column prop="origin_url" label="Original URL"></el-table-column>
      <el-table-column prop="short_url" label="Short URL"></el-table-column>
      <el-table-column prop="expire_at" label="Expire At"></el-table-column>
      <el-table-column prop="creat_at" label="Created At"></el-table-column>
      <el-table-column prop="clicks" label="Clicks"></el-table-column>
      <el-table-column prop="last_click_time" label="Last Click Time"></el-table-column>
    </el-table>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }


}
</style>
