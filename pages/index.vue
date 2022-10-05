<template>
  <div>
      <input
        type="text"
        name="searchInput"
        id="searchInput"
        v-model="searchInput"
        placeholder="Enter a name"
      >

      <template v-for="pokemon in filterPokemon">
        <div :key="pokemon.name" v-if="filterPokemon.length < 30">
          <button
            type="button"
            @click.prevent="buttonPokemon(pokemon.name)"
          >
            {{ pokemon.name }}
          </button>
        </div>
      </template>

      <template>
        <h1>{{ choosenPokemon }}</h1>
        <div v-for="(ability, index) in pokemonInfo.abilities" :key="index">
          {{ ability }}
        </div>
      </template>
  </div>
</template>

<script>
  import axios from 'axios';

  export default {
    name: 'IndexPage',
    data() {
      return {
        pokeApi: [],
        searchInput: '',
        pokemonInfo: {},
        choosenPokemon: '',
      }
    },
    async fetch() {

      await axios
        .get('https://pokeapi.co/api/v2/pokemon/?offset=0&limit=2000')
        .then( resp => this.pokeApi = resp.data.results)
        .catch( error => console.log(error))

    },
    methods: {
      buttonPokemon(value) {
        console.log(value);
        this.choosenPokemon = value

        axios
          .get(`https://pokeapi.co/api/v2/pokemon/${this.choosenPokemon}`)
          .then( resp => (
            this.pokemonInfo = resp.data,
            console.log(resp.data)
          ) )
          .catch( error => console.log(error))

        return this.pokemonInfo
      },
    },
    computed: {
      filterPokemon() {
        return this.pokeApi.filter( 
          pokemon => {
            return pokemon.name.includes(this.searchInput)
          }) 
      },
    },
  }
</script>
