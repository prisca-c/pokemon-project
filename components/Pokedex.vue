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
    <b-row v-if="filteredPokemonList.length < 30" class="d-flex justify-content-center mx-1 my-3">
      <div v-for="pokemon in filteredPokemonList" :key="pokemon.name" class="p-1">
        <b-button
          type="button"
          @click.prevent="pickPokemon(pokemon.name)"
          class="text-capitalize btn-pokemon"
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
          <span class="text-capitalize">{{ choosenPokemon }}</span>
        </h2>

        <!-- Display Pokemon's Image -->
        <div
          v-if="choosenPokemon"
          class="mx-auto mb-4 mt-3"
        >
          <!-- Component which render the choosen pokemon's image/sprite -->
          <sprite :pokemon="choosenPokemon" :key="choosenPokemon" />
        </div>

        <!-- Display Evolution's infos -->
        <div
          class="d-flex flex-column justify-content-center flex-wrap"
          v-if="filteredEVOLUTION"
        >
          <h2 class="text-capitalize text-center"
              v-if="filteredEVOLUTION.length > 1">
            {{ choosenPokemon }}'s evolutions
          </h2>

          <div
            class="d-flex justify-content-center flex-wrap"
            v-if="filteredEVOLUTION.length > 1"
          >
            <!-- Get evo's image/sprite -->
            <sprite
              v-for="evolution in filteredEVOLUTION"
              :pokemon="evolution.species_name"
              :key="evolution.species_name"
              getHeight="100"
              getWidth="100"
              :label="evolution.species_name"
              class="mx-3"
            />
          </div>

        </div>

        <!-- Display Flavors Text -->
        <p v-if="getSpecies.flavor_text_entries" class="text-center m-auto">
          {{ filteredFLAVORTEXTlang[0].flavor_text.replace(/\u000c/g, ' ') }}
        </p>

        <!-- Display abilities -->
        <h2>Abilities</h2>
        <div v-if="getAbilities.length > 0">

          <b-card class="border-radius-sm shadow-sm">
            <b-tabs>
              <b-tab
                card-header
                tabs
                v-for="ability in getAbilities"
                :key="ability.name"
                :title="`#${ability.id} - ${ability.name.toUpperCase()}`"
              >

                <!-- Display effect -->
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

                <!-- Display pokemon's with the same abillity -->
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
      getSpecies: {},
      getEVOLUTIONCHAIN: {},
      getEvolutionLoopData: {},
      componentKey: '',
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
      this.getPokemonSPECIES()
      this.forceRerenderer()
    },
    pickPokemon(value) {
      this.choosenPokemon = value
      
      axios
        .get(`https://pokeapi.co/api/v2/pokemon/${this.choosenPokemon}`)
        .then( resp => { 
          
          this.pokemonInfo = resp.data

          this.init()
          
        })
        .catch( error => console.log(error))
      
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
    },
    getPokemonSPECIES() {
      if (this.pokemonInfo.species){

        axios
          .get(this.pokemonInfo.species.url)
          .then( 
            resp => {

              this.getSpecies = resp.data

              this.getPokemonEVOLUTIONCHAIN()
          })
          .catch( error => console.log(error))
          
      }
    },
    getPokemonEVOLUTIONCHAIN() {
      if (this.getSpecies.evolution_chain){
        axios
          .get(this.getSpecies.evolution_chain.url)
          .then(
            resp => {
              this.getEVOLUTIONCHAIN = resp.data
            }
          )
          .catch( error => console.log(error))
      }
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
    /* getPokemonSPRITE(pokemon) {
      let sprites = {}
      {
        axios
          .get(`https://pokeapi.co/api/v2/pokemon/${pokemon}`)
          .then(
            resp => {
              sprites = resp.data.sprites
              console.log(sprites);
            }
          )
          .catch(error => console.log(error))
        
        return sprites
      }
    }, */
  },
  computed: {
    filteredPokemonList() {
      return this.pokeApi.filter( 
        pokemon => pokemon.name.includes(this.searchInput)
      )
    },
    filteredFLAVORTEXT() {
      if(this.getSpecies.flavor_text_entries) {
        return this.getSpecies.flavor_text_entries.slice([0], [10]).map((item) => {
          return ({flavor_text: item.flavor_text, language: item.language.name});
        })
      }
    },
    filteredFLAVORTEXTlang() {
      if(this.getSpecies.flavor_text_entries) {
        return this.filteredFLAVORTEXT.filter(
          text => text.language === 'en'
        ).slice([0], [1])
      }
    },
    filteredEVOLUTION() {
      if (this.getEVOLUTIONCHAIN.chain) {
        let evoChain = [];
        let evoData = this.getEVOLUTIONCHAIN.chain;

        do {
            let numberOfEvolutions = evoData['evolves_to'].length;
            let evoDetails = evoData['evolution_details'][0];
            
            if(numberOfEvolutions <= 1){
              evoChain.push({
                "species_name":
                  evoData .species.name,
                "min_level":
                  !evoDetails
                    ? 1
                    : evoDetails .min_level,
                "trigger_name":
                  !evoDetails
                    ? null
                    : evoDetails .trigger.name,
                "item":
                  !evoDetails
                    ? null
                    : evoDetails .item,
                "min_affection":
                  !evoDetails
                    ? null
                    : evoDetails .min_affection,
                "time_of_day":
                  !evoDetails
                    ? null
                    : evoData .time_of_day,
              })
            };

            // Needed for all Eevee's evolutions...
            if(numberOfEvolutions > 1) {
              for (let i = 1;i < numberOfEvolutions; i++) { 
                evoChain.push({
                  "species_name":
                    evoData.evolves_to[i].species.name,
                  "min_level": 
                    !evoData.evolves_to[i]
                      ? 1
                      : evoData.evolves_to[i].evolution_details[0].min_level,
                  "trigger_name": 
                    !evoData.evolves_to[i]
                      ? null 
                      : (
                          // Ternary needed to deal with multiple case of evolutions,
                          // i choose to deal with evolution stone's one
                          evoData.evolves_to[i].evolution_details.length === 5
                            ? evoData.evolves_to[i].evolution_details[4].trigger.name
                            : evoData.evolves_to[i].evolution_details[0].trigger.name
                        ),
                  "item":
                    !evoData.evolves_to[i]
                      ? null
                      : (
                          // Ternary needed to deal with multiple case of evolutions,
                          // i choose to deal with evolution stone's one
                          evoData.evolves_to[i].evolution_details.length === 5
                            ? evoData.evolves_to[i].evolution_details[4].item
                            : evoData.evolves_to[i].evolution_details[0].item
                        ),
                  "min_affection":
                    !evoData.evolves_to[i]
                      ? null
                      : evoData.evolves_to[i].evolution_details[0].min_affection,
                  "time_of_day":
                    !evoData.evolves_to[i]
                      ? null
                      : evoData.evolves_to[i].evolution_details[0].time_of_day,
              });}}        

            evoData = evoData.evolves_to[0];
          
        } while (evoData != undefined && evoData.hasOwnProperty('evolves_to'));

        return evoChain; 
      }
    },
  },
}
</script>

<style scoped>

.pokemonName {
  line-height: 30px;
}

.btn-pokemon{
  width: 218px;
}

</style>