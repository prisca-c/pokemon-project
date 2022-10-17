<template>
  <div class="m-2">

    <!-- Search bar -->
    <div class="mw-900 m-auto">
      <b-form-input
        type="text"
        name="searchInput"
        id="searchInput"
        v-model="searchInput"
        placeholder="Enter a name"
        autofocus
      />
      <!-- Pick a pokemon -->
      <b-row
        v-if="filteredPokemonList.length < 30"
        class="mw-900 d-flex justify-content-around my-3 mx-auto"
      >
        <div v-for="pokemon in filteredPokemonList" :key="pokemon.name">
          <b-button
            type="button"
            @click.prevent="pickPokemon(pokemon.name)"
            class="text-capitalize btn-pokemon"
          >
            {{ pokemon.name }}
          </b-button>
        </div>
      </b-row>
    </div>

    <!-- Display Pokémon's infos -->
    <template v-if="choosenPokemon">
      <div class="d-flex flex-column">
        <h2 class="pokemonName mt-4 text-center font-weight-bold">
          #{{ pokemonInfo.id }} 
          <br>
          <span class="text-capitalize">{{ choosenPokemon }}</span>
        </h2>

        <!-- Display Pokemon's Image -->
        <div
          v-if="choosenPokemon"
          class="mx-auto mb-4 mt-1"
        >
          <!-- Component which render the choosen pokemon's image/sprite -->
          <sprite :pokemon="choosenPokemon" :key="choosenPokemon" />
        </div>

        <!-- Display Evolution's infos -->
        <div
          class="d-flex flex-column justify-content-center flex-wrap"
          v-if="filteredEVOLUTION"
        >
          <h2 class="text-capitalize text-center font-weight-bold"
              v-if="filteredEVOLUTION.length > 1">
            {{ choosenPokemon }}'s evolutions
          </h2>

          <div
            class="d-flex justify-content-center flex-wrap mb-4"
            v-if="filteredEVOLUTION.length > 1"
          >
            <div
              class="evolution-box"
              v-for="evolution in filteredEVOLUTION"
              :key="`${evolution.species_name}'-evo'`"
            >
              <!-- Get evo's image/sprite -->
              <sprite
                :pokemon="evolution.species_name"
                getHeight="100"
                getWidth="100"
                :label="evolution.species_name"
                class="mx-3"
              />
              <div
                class="evolution-infos text-capitalize"
              >
                <!-- Display basic evolution -->
                <p
                  v-if="
                    evolution.min_level !== null
                  "
                  class="text-center lh-20"
                >
                  <span class="font-weight-bold">
                    Level <br>
                  </span>

                  {{ evolution.min_level }}
                </p>
                
                <!-- Display Day/Night evolution -->
                <p
                  class="text-center  lh-20"
                  v-else-if="
                    evolution.time_of_day === 'day' ||
                    evolution.time_of_day === 'night'
                  "
                >

                  <span
                    class="font-weight-bold"
                  >
                    Evoltve at<br>
                  </span>

                  {{ evolution.time_of_day }}

                </p>

                <!-- Display Min Happiness evolution -->
                <p
                  class="text-center lh-20"
                  v-else-if="
                    evolution.min_happiness !== null &&
                    evolution.min_level === null
                  "
                >
                  <span
                    class="font-weight-bold"
                  >
                    Happinness <br>
                  </span>

                  <span style="color: #ff8cb8">{{ evolution.min_happiness }}</span>
                </p>

                <!-- Display Trade evolution -->
                <p
                  v-else-if="evolution.trigger_name === 'trade'"
                  class="text-center lh-20"
                >
                  <span class="font-weight-bold">Trade</span>
                </p>

                <!-- Display Stone Evolution -->
                <p
                  v-else-if="evolution.trigger_name === 'use-item'"
                  class="text-center lh-20"
                >
                  <span class="font-weight-bold">
                    Stone Evo'
                  </span>

                  <span :style="{ color: getStoneColor(evolution.item.name),}">
                    <br>{{ evolution.item.name }}
                  </span>
                </p>

                <!-- Display Affection evolution -->
                <p
                  class="text-center lh-20"
                  v-else-if="evolution.min_affection"
                >
                  <span
                    class="font-weight-bold"
                  >
                    Affection <br>
                  </span>

                  <span style="color: #ff8cb8">{{ evolution.min_affection }}</span>
                </p>

                <!-- Display if no informations -->
                <p v-else class="text-center lh-20 font-weight-bold">
                  No
                  <br>Informations
                </p>

              </div>
            </div>
          </div>

        </div>

        <!-- Display Flavors Text -->
        <p v-if="getSpecies.flavor_text_entries" class="mw-900 text-center m-auto">
          {{ filteredFLAVORTEXTlang[0].flavor_text.replace(/\u000c/g, ' ') }}
        </p>

        <!-- Display abilities -->
        <div
          v-if="getAbilities.length > 0"
          class="mw-900 mx-auto mt-4"
        >
          <h2>Abilities</h2>

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
      stoneColor: "",
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
      //this.forceRerenderer()
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
    getStoneColor(stone) {
      if (this.getSpecies.evolution_chain){
        if (stone === "thunder-stone") return "#ebd700"
        if (stone === "fire-stone") return "#F09030"
        if (stone === "leaf-stone") return "#88D088"
        if (stone === "ice-stone") return "#7CC9D6"
        if (stone === "moon-stone") return "#556D5D"
        if (stone === "water-stone") return "#8080E4"
        if (stone === "sun-stone") return "#CC6734"
        if (stone === "shiny-stone") return "#EDEF8F"
        if (stone === "dusk-stone") return "#705078"
        if (stone === "dawn-stone") return "#18B0A0"
      }
    },
    checkIfPokemonExist(value) {
      let checkPokemon = []
      
      axios
        .get(`https://pokeapi.co/api/v2/pokemon/${value}`)
        .then( resp => checkPokemon.push(resp.data))
        .catch(error => error)
      
      return checkPokemon
    }
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
                "min_happiness":
                  !evoDetails
                    ? null
                    : evoDetails .min_happiness,
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
                  "min_happiness":
                    !evoData.evolves_to[i]
                      ? null
                      : evoData.evolves_to[i].evolution_details[0].min_happiness,
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
  margin-bottom: 8px;
}

.mw-900{
  max-width: 900px;
}

.lh-20 {
  line-height: 20px;
}

</style>