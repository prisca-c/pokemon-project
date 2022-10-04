<template>
  <div>
    {{ pokeApi.abilities }}
  </div>
</template>

<script>
  export default {
    name: 'IndexPage',
    data() {
      return {
        pokeApi: [],
      }
    },
    async fetch() {
      const axios = require('axios');

      this.pokeApi = axios.get('https://pokeapi.co/api/v2/pokemon/?offset=0&limit=2000')
        .then( resp => console.log(resp.data))
        .catch( error => console.log(error))
    }
  }
</script>
