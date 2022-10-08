<template>
  <div>

    <!-- Search bar -->
    <input
      type="text"
      name="searchInput"
      id="searchInput"
      v-model="searchInput"
      placeholder="Enter a name"
    >

    <!-- Pick a pokemon -->
    <b-row v-if="filteredPokemonList.length < 30">
      <div v-for="pokemon in filteredPokemonList" :key="pokemon.name" class="p-1">
        <b-button
          type="button"
          @click.prevent="pickPokemon(pokemon.name)"
          class="capitalize"
        >
          {{ pokemon.name }}
        </b-button>
      </div>
    </b-row>

    <!-- Display Pokémon's infos -->
    <template v-if="choosenPokemon">
      <h2>#{{ pokemonInfo.id }} - <span class="capitalize">{{ choosenPokemon }}</span></h2>
      <div>

        <!-- Section Abilities -->
        <h2>Abilities</h2>
          <div v-if="getAbilities.length > 0">

            <template>
              <b-card class="border-radius-sm shadow-sm m-2">
                <b-tabs>
                    <b-tab
                      card-header
                      tabs
                      v-for="ability in getAbilities"
                      :key="ability.name"
                      :title="ability.name.toUpperCase()"
                      >
                      <div v-if="ability.effect_entries.length > 1">
                        <h3>Effect</h3>
                        <p>{{ ability.effect_entries[1].effect }}</p>
                      </div>

                      <div v-else-if="ability.effect_entries.length === 1">
                        <h3>Effect</h3>
                        <p>{{ ability.effect_entries[0].effect }}</p>
                      </div>
                      
                      <div v-else>
                        <h3>Effect</h3>
                        <p>No information</p>
                      </div>
                    </b-tab>
                </b-tabs>
              </b-card>
            </template>
            
          </div>
      </div>
    </template>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'Pokedex',

  data() {
    return {
      pokeApi: [],
      searchInput: '',
      pokemonInfo: {},
      choosenPokemon: '',
      getAbilities: [],
    }
  },
  async fetch() {

    const resp = await axios
      .get('https://pokeapi.co/api/v2/pokemon/?offset=0&limit=2000')
    
    this.pokeApi = resp.data.results

  },
  methods: {
    init() {
      this.getPokemonABILITIES()
    },
    pickPokemon(value) {
      console.log(value);
      this.choosenPokemon = value
      {
      axios
        .get(`https://pokeapi.co/api/v2/pokemon/${this.choosenPokemon}`)
        .then( resp => { 

          this.pokemonInfo = resp.data
          console.log(this.pokemonInfo)

          this.init()
          })
          .catch( error => console.log(error))
      }

      
    },
    getPokemonABILITIES() {
      if (this.pokemonInfo.abilities){
        this.getAbilities = []
        this.pokemonInfo.abilities.forEach(
          (ability) => {

            axios
              .get(`${ability.ability.url}` )
              .then( resp => {
                this.getAbilities.push(resp.data)
          })}
      )}
      console.log(this.getAbilities)
    },
  },
  computed: {
    filteredPokemonList() {
      return this.pokeApi.filter( 
        pokemon => pokemon.name.includes(this.searchInput)
      )
    },
    filteredABILITIES(path) {
      return path.find(
        ability => ability.includes("en")
      )
    },
  },
}
</script>

<style>

.capitalize {
  text-transform: capitalize;
}

</style>