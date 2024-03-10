<script setup>
import { ref } from 'vue'
import axios from 'axios'
let Long_Url = ref('')
let Shorted_Url = ref('')
let DisplayInformation = ref('')
let Error_message = ref('')
let Target_Url = ref('')
axios.defaults.headers.post['Content-Type'] = 'application/json';
axios.defaults.baseURL = 'http://localhost:8080';
//shorten the url
function shortenUrl() {
  if (isUrl(Long_Url.value)) {
    Error_message.value = '';
    // Post the URL to the http://localhost:8080/api/links, and get the shortened URL
    axios.post(`/api/links?longUrl=${Long_Url.value}`).then(response => {
      console.log(response.data);
      // 注意这里修正了URL的构建方式
      Shorted_Url.value = axios.defaults.baseURL +"/"+response.data.shortUrl;
    }).catch(error => {
      // 这里你可以考虑使用 error.message 获取更具体的错误信息
      Error_message.value = error.message;
    });
  } else {
    Error_message.value = 'Please input a valid URL.';
  }
}
//display all information
function displayAll() {
  axios.get(`/api/logs?target=all`).then(response => {
    console.log(response.data);
    DisplayInformation.value = response.data;
  }).catch(error => {
    console.log(error);
  });
}
function displayTargetInfomation() {
  axios.get(`/api/logs?target=${isShortedUrl(Target_Url.value)}`).then(response => {
    console.log(response.data);
    DisplayInformation.value = response.data;
  }).catch(error => {
    console.log(error);
  });
}
//check if the url is shorted
function isShortedUrl(str) {
  let pattern = new RegExp('^' + axios.defaults.baseURL + '/[a-zA-Z0-9]{6}$');
  if (pattern.test(str)) {
    return str.slice(-6);
  } else {
    return str;
  }
}
//check if the url is valid
function isUrl(str) {
  let pattern = new RegExp('^(https?:\\/\\/)?'+ // protocol
    '((([a-z\\d]([a-z\\d-]*[a-z\\d])*)\\.)+[a-z]{2,}|'+ // domain name
    '((\\d{1,3}\\.){3}\\d{1,3}))'+ // OR ip (v4) address
    '(\\:\\d+)?(\\/[-a-z\\d%_.~+]*)*'+ // port and path
    '(\\?[;&a-z\\d%_.~+=-]*)?'+ // query string
    '(\\#[-a-z\\d_]*)?$','i'); // fragment locator
  return !!pattern.test(str);
}

</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

  </header>

  <main>
    <p>please input the url you need to shortener</p>
    <input v-model="Long_Url" placeholder="origin url">
    <button @click="shortenUrl">shorten</button>
    <p v-if="Error_message">{{Error_message}}</p>
    <a :href="Shorted_Url" v-if="Shorted_Url">shortened url:{{Shorted_Url}}</a>
    <p></p>
    <button @click="displayAll">display all information</button>
    <input v-model="Target_Url" placeholder="Target Url">
    <button @click="displayTargetInfomation">display target information</button>
    <p v-if="DisplayInformation">all information:{{ DisplayInformation }}</p>
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

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
