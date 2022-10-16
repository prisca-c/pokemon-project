<template>
  <div
  v-if="sprites.length > 0"
    class="
      pokemonImage mb-4 mt-3 d-inline-block p-2 border border-secondary rounded-lg"
  >
    <img
      v-if="sprites[0].other.home.front_default !== null"
      :src="sprites[0].other.home.front_default"
      :alt="`${pokemon}'s illustration`"
      :height="getHeight"
      :width="getWidth"
    >

    <img
      v-else-if="sprites[0].other.home.front_default === null && sprites[0].front_default !== null"
      :src="sprites[0].front_default"
      :alt="`${pokemon}'s illustration`"
      :height="getHeight"
      :width="getWidth"
    >

    <div v-else class="text-center noIllustration">
      <p class="m-auto">No Illustration Available</p>
    </div>

    <p class="text-capitalize text-center mt-2">{{ label }}</p>
  </div>
</template>

<script>
  import axios from 'axios';
  
  export default {
    name: 'Sprite',
    props: {
      pokemon: {
        type: String,
        default: null,
        required: true,
      },
      getHeight: {
        type: Number,
        default: 200,
        required: false,
      },
      getWidth: {
        type: Number,
        default: 200,
        required: false,
      },
      label: {
        type: String,
        default: '',
        required: false,
      }
    },
    data() {
      return {
        sprites: [],
      }
    },
    created() {
      this.getPokemonSPRITE()
    },
    methods: {
      getPokemonSPRITE() {
        {
          axios
            .get(`https://pokeapi.co/api/v2/pokemon/${this.pokemon}`)
            .then(
              resp => {
                this.sprites.push(resp.data.sprites)
              }
            )
            .catch(error => console.log(error))
            
        }
      },
    },
  }
</script>

<style>

.noIllustration {
  width: 200px;
  height: 200px;
}

</style>