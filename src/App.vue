<template>
  <div class="title-app">
    <h2 class="title">POKÉDEX INTERNACIONAL</h2>
  </div>
  <div class="search-container">
    <input type="text" v-model="searchQuery" placeholder="Buscar Pokémon..." class="search-input"
      @keyup.enter="searchPokemon">
    <button @click="searchPokemon" class="btn btn-primary">Buscar</button>
  </div>
  <div class="pokemon-list">
    <div class="pagination">
      <button @click="firstPage" :disabled="offset === 0 || loading" class="btn btn-secondary">Inicio</button>
      <button @click="previousPage" :disabled="offset === 0 || loading" class="btn btn-secondary">Anterior</button>
      <button @click="nextPage" :disabled="offset + limit >= totalCount || loading"
        class="btn btn-secondary">Siguiente</button>
      <button @click="lastPage" :disabled="offset + limit >= totalCount || loading"
        class="btn btn-secondary">Final</button>
    </div>
    <div v-if="loading" class="pokeball">
      <div class="pokeball_detalle"></div>
    </div>
    <div v-else class="pokemon-container">
      <div v-for="pokemon in visiblePokemon" :key="pokemon.id" class="pokemon-card" @click="toggleDetails(pokemon)">
        <img :src="pokemon.imageUrl" :alt="pokemon.name" class="pokemon-image" @error="handleImageError(pokemon)" />
        <div class="pokemon-info">
          <h6 class="pokemon-id"># {{ pokemon.id }}</h6>
          <h5 class="pokemon-name">{{ pokemon.name }}</h5>
          <div v-if="pokemon.showDetails" class="pokemon-details">
            <div>Tipo(s): {{ pokemon.types.join(', ') }}</div>
            <div>Habilidades: {{ pokemon.abilities.join(', ') }}</div>
          </div>
        </div>
      </div>
    </div>
    <div class="pagination">
      <button @click="firstPage" :disabled="offset === 0 || loading" class="btn btn-secondary">Inicio</button>
      <button @click="previousPage" :disabled="offset === 0 || loading" class="btn btn-secondary">Anterior</button>
      <button @click="nextPage" :disabled="offset + limit >= totalCount || loading"
        class="btn btn-secondary">Siguiente</button>
      <button @click="lastPage" :disabled="offset + limit >= totalCount || loading"
        class="btn btn-secondary">Final</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { ref } from 'vue';

export default {
  data() {
    return {
      loading: false,
      limit: 50,
      offset: 0,
      totalCount: 0,
      pokemonData: [],
      visiblePokemon: [],
      searchQuery: ''
    };
  },
  created() {
    this.fetchPokemonData();
  },
  methods: {
    fetchPokemonData() {
      this.loading = true;
      axios.get(`https://pokeapi.co/api/v2/pokemon?limit=${this.limit}&offset=${this.offset}`)
        .then(response => {
          this.totalCount = response.data.count;
          return Promise.all(response.data.results.map(pokemon => this.fetchPokemonDetails(pokemon.url)));
        })
        .then(pokemons => {
          this.pokemonData = pokemons;
          this.visiblePokemon = this.pokemonData;
          this.loading = false;
        })
        .catch(error => {
          console.error('Error al obtener información de los Pokémon:', error);
          this.loading = false;
        });
    },
    fetchPokemonDetails(url) {
      return axios.get(url)
        .then(response => {
          const pokemon = response.data;
          let imageUrl = pokemon.sprites.front_default;
          if (!imageUrl) {
            // Si no hay una imagen disponible, utilizar una imagen de error predeterminada
            imageUrl = require("./assets/error.png")
          }
          return {
            id: pokemon.id,
            name: pokemon.name,
            imageUrl: imageUrl,
            types: pokemon.types.map(type => type.type.name),
            abilities: pokemon.abilities.map(ability => ability.ability.name),
            showDetails: false
          };
        });
    },
    firstPage() {
      this.offset = 0;
      this.fetchPokemonData();
    },
    previousPage() {
      if (this.offset >= this.limit) {
        this.offset -= this.limit;
        this.fetchPokemonData();
      }
    },
    nextPage() {
      if (this.offset + this.limit < this.totalCount) {
        this.offset += this.limit;
        this.fetchPokemonData();
      }
    },
    lastPage() {
      const lastPageOffset = Math.max(0, this.totalCount - this.limit);
      if (this.offset !== lastPageOffset) {
        this.offset = lastPageOffset;
        this.fetchPokemonData();
      }
    },
    toggleDetails(pokemon) {
      pokemon.showDetails = !pokemon.showDetails;
    },
    searchPokemon() {
      if (this.searchQuery.trim() === '') {
        this.fetchPokemonData();
      } else {
        this.loading = true;
        const searchTerm = this.searchQuery.trim().toLowerCase();
        axios.get(`https://pokeapi.co/api/v2/pokemon/${searchTerm}`)
          .then(response => {
            const pokemon = response.data;
            const pokemonDetails = {
              id: pokemon.id,
              name: pokemon.name,
              imageUrl: pokemon.sprites.front_default,
              types: pokemon.types.map(type => type.type.name),
              abilities: pokemon.abilities.map(ability => ability.ability.name),
              showDetails: false
            };
            this.visiblePokemon = [pokemonDetails];
            this.loading = false;
          })
          .catch(error => {
            console.error('Error al obtener información del Pokémon:', error);
            this.loading = false;
          });
      }
    }
  }
};
</script>

<style scoped>
.pokemon-list {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #c01c1d;
}

.pokemon-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;
  /* Espacio entre las tarjetas */
}

.pokemon-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: white;
  border: 2px solid #000000;
  border-radius: 5px;
  padding: 10px;
}

.pokemon-image {
  border: 2px solid #000000;
  border-radius: 5px;
  width: 100px;
  height: 100px;
  margin-bottom: 10px;
  background: linear-gradient(#608d91, #beeef8, #abe1ee, #6fb7c0);
}

.pokemon-info {
  text-align: center;
  background-color: #1e3336;
  color: #b5c1c3;
  border-radius: 5px;
}

.pagination {
  margin-top: 20px;
  margin-bottom: 20px;
}

/* Estilos para los detalles del Pokémon */
.pokemon-details {
  margin-top: 10px;
  margin-left: 5px;
  margin-right: 5px;
  margin-bottom: 5px;
}

.pokemon-name {
  text-transform: uppercase;
  margin-left: 5px;
  margin-right: 5px;
}

.title {
  margin-top: 30px;
  color: #ffffff;
  text-shadow: 2px 2px 5px black;
}

.title-app {
  background-color: #801416;
  min-width: 100%;
  padding: 5px;
  text-align: center;
}

.btn {
  margin-left: 10px;
  margin-right: 10px;
}

.pokeball {
  width: 10rem;
  height: 10rem;
  background-color: #ffffff;
  border-radius: 50%;
  position: relative;
  animation: spin 2s linear infinite;
}

.pokeball::before {
  content: '';
  width: 10rem;
  height: 5rem;
  background-color: #ff1c1c;
  border-radius: 5rem 5rem 0 0;
  position: absolute;

}

.pokeball_detalle {
  width: 10rem;
  height: .5rem;
  background-color: #333;
  margin-top: 4.8rem;
  position: absolute;

}

.pokeball_detalle::before {
  content: '';
  width: 3rem;
  height: 3rem;
  background-color: #333;
  border-radius: 50%;
  position: absolute;
  margin-top: -1.1rem;
  margin-left: 3.5rem;
}

.pokeball_detalle::after {
  content: '';
  width: 1.8rem;
  height: 1.8rem;
  background-color: #fff;
  position: absolute;
  border-radius: 50%;
  margin-left: 4.1rem;
  margin-top: -0.5rem;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}

.search-container {
  background-color: #c01c1d;
  display: flex;
  width: 100%;
  padding: 10px;
}

.search-input {
  flex: 1;
  margin-right: 10px;
  background-color: #33cb66;
  color: white;
  border: none;
  padding: 5px;
  border: 2px solid #000000;

}

.btn-primary {
  background-color: #3297cb;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}

.search-input::placeholder {
  color: white;
  /* Cambiar el color del placeholder a blanco */
}
</style>
