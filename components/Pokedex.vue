<template>
  <div class="m-2">

    <!-- Search bar -->
    <b-form-input
      type="text"
      name="searchInput"
      id="searchInput"
      v-model="searchInput"
      placeholder="Enter a name"
      autofocus
    />

    <!-- Pick a pokemon -->
    <b-row v-if="filteredPokemonList.length < 30" class="mx-1 my-3">
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
      <div class="d-flex flex-column">
        <h2 class="pokemonName mt-4 text-center">
          #{{ pokemonInfo.id }} 
          <br>
          <span class="capitalize">{{ choosenPokemon }}</span>
        </h2>
        <!-- Display Pokemon's Image -->
        <div
          v-if="pokemonInfo.sprites"
          class="
            pokemonImage mb-4 mt-3 d-inline-block p-2 border border-secondary rounded-lg"
        >
          <img
            v-if="pokemonInfo.sprites.other.home.front_default !== null"
            :src="pokemonInfo.sprites.other.home.front_default"
            :alt="`${choosenPokemon}'s illustration`"
            height="200px"
            width="200px"
          >
          <img
            v-else-if="pokemonInfo.sprites.other.home.front_default === null && pokemonInfo.sprites.front_default !== null"
            :src="pokemonInfo.sprites.front_default"
            :alt="`${choosenPokemon}'s illustration`"
            height="200px"
            width="200px"
          >
          <div v-else>
            <p>No Illustration Available</p>
          </div>

        </div>

          <p v-if="getSpecies.length > 0" class="text-center m-auto">
            {{ getSpecies[0].flavor_text_entries[0].flavor_text.replace(/\u000c/g, ' ')}}
          </p>

        <h2>Abilities</h2>
        <div v-if="getAbilities.length > 0">

          <template>
            <b-card class="border-radius-sm shadow-sm">
              <b-tabs>
                <b-tab
                  card-header
                  tabs
                  v-for="ability in getAbilities"
                  :key="ability.name"
                  :title="`#${ability.id} - ${ability.name.toUpperCase()}`"
                >
                  
                  <div class="effect">
                    <div v-if="ability.effect_entries.length > 0">
                      <h3>Effect</h3>
                      <p>
                        {{ filteredLangEFFECT(ability.effect_entries).effect }}
                      </p>
                    </div>
                    <div v-else>
                      <h3>Effect</h3>
                      <p>No informations</p>
                    </div>
                  </div>

                  <div v-if="ability.pokemon.length > 0">
                    <b-dd
                      id="dropdown-buttons dropdown-offset"
                      text="Pokemon with this abillity"
                      class="mt-2"
                      block
                      no-flip
                    >
                      <b-dd-item-button
                        v-for="pokemon in ability.pokemon"
                        :key="pokemon.pokemon.name"
                        @click.prevent="pickPokemon(pokemon.pokemon.name)"
                        class="d-inline-block text-center w-50 my-2"
                      >
                        {{ pokemon.pokemon.name }}
                      </b-dd-item-button>
                    </b-dd>
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
      getSpecies: [],
    }
  },
  async fetch() {

    const resp = await axios
      .get('https://pokeapi.co/api/v2/pokemon/?offset=0&limit=2000')
    
    this.pokeApi = resp.data.results

  },
  methods: {
    init() {
      this.getPokemonSPECIES() 
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
              })
              .catch( error => console.log(error))
          }
      )}
      console.log(this.getAbilities)
    },

    /** 
     * Needed to get only English effect entries
     * ( because sometimes language's arrays are reversed ,
     * can't pick one with only index in v-for )
    */
    filteredLangEFFECT(path) {
      return path.find(
        ability => ability.language.name === "en"
      )
    },
    getPokemonSPECIES() {
      if (this.pokemonInfo.species){
        this.getSpecies = []
        axios
          .get(`${this.pokemonInfo.species.url}`)
          .then( 
            resp => {
              this.getSpecies.push(resp.data)
          })
          .catch( error => console.log(error))
        }
    }
  },
  computed: {
    filteredPokemonList() {
      return this.pokeApi.filter( 
        pokemon => pokemon.name.includes(this.searchInput)
      )
    },
  },
}
</script>

<style>

.capitalize {
  text-transform: capitalize;
}

.pokemonName {
  line-height: 30px;
}

.pokemonImage {
  margin-left: auto;
  margin-right: auto;
}

</style>