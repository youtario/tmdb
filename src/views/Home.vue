<script setup>
import { ref, onMounted, watch, computed } from 'vue';
import Loading from 'vue-loading-overlay';
import api from '@/plugins/axios';

const movies = ref([]);
const genres = ref([]);
const selectedGenre = ref(null);
const selectedMovie = ref(null);
const isLoading = ref(false);
const searchTerm = ref('');
const sortBy = ref('popularity.desc');

const filteredMovies = computed(() => {
  return movies.value.filter(movie =>
    movie.title.toLowerCase().includes(searchTerm.value.toLowerCase())
  );
});

const fetchGenres = async () => {
  try {
    const response = await api.get('genre/movie/list?language=pt-BR');
    genres.value = response.data.genres;
  } catch (error) {
    console.error('Erro ao buscar gêneros:', error);
  }
};

const fetchMovies = async () => {
  isLoading.value = true;
  try {
    const params = {
      language: 'pt-BR',
      sort_by: sortBy.value,
      'vote_average.gte': 5,
      'vote_average.lte': 7,
      'vote_count.gte': 100,
    };

    if (selectedGenre.value) {
      params.with_genres = selectedGenre.value;
    }

    const response = await api.get('discover/movie', { params });
    movies.value = response.data.results || [];
  } catch (error) {
    console.error('Erro ao buscar filmes:', error);
  } finally {
    isLoading.value = false;
  }
};

const fetchMovieDetails = async (movieId) => {
  try {
    const response = await api.get(`movie/${movieId}?language=pt-BR`);
    selectedMovie.value = response.data;
  } catch (error) {
    console.error('Erro ao buscar detalhes:', error);
  }
};

const getGenreName = (id) => {
  return genres.value.find(g => g.id === id)?.name || '';
};

const formatDate = (date) => {
  if (!date) return 'Data desconhecida';
  return new Date(date).toLocaleDateString('pt-BR');
};

const selectGenre = (genreId) => {
  selectedGenre.value = genreId;
};

const closeModal = () => {
  selectedMovie.value = null;
};

onMounted(async () => {
  await fetchGenres();
  await fetchMovies();
});

watch([selectedGenre, sortBy], () => {
  fetchMovies();
});
</script>

<template>
  <div class="app-container">
    <nav class="main-nav">
      <router-link to="/descoberta">Descoberta</router-link>
    </nav>

    <header class="header">
      <div class="header-content">
        <div class="logo">
          <div>
            <h1 class="logo-title">Na Medida</h1>
            <p class="logo-subtitle">A beleza do meio-termo (5-7)</p>
          </div>
        </div>
        <div class="search-box">
          <span class="search-icon"></span>
          <input
            type="text"
            placeholder="Buscar filmes..."
            v-model="searchTerm"
            class="search-input"
          />
        </div>
      </div>
    </header>

    <div class="filters-section">
      <div class="filter-group">
        <span class="filter-icon">⚙️</span>
        <span class="filter-label">Ordenar:</span>
        <select v-model="sortBy" class="select">
          <option value="popularity.desc">Mais populares</option>
          <option value="release_date.desc">Mais recentes</option>
          <option value="vote_average.desc">Melhor avaliados</option>
          <option value="vote_count.desc">Mais votados</option>
        </select>
      </div>
    </div>

    <div class="genres-container">
      <button
        @click="selectGenre(null)"
        class="genre-pill"
        :class="{ active: selectedGenre === null }"
      >
        Todos
      </button>
      <button
        v-for="genre in genres"
        :key="genre.id"
        @click="selectGenre(genre.id)"
        class="genre-pill"
        :class="{ active: selectedGenre === genre.id }"
      >
        {{ genre.name }}
      </button>
    </div>

    <loading v-model:active="isLoading" is-full-page />

    <div v-if="!isLoading" class="movies-grid">
      <div
        v-for="movie in filteredMovies"
        :key="movie.id"
        class="movie-card"
        @click="fetchMovieDetails(movie.id)"
      >
        <div class="poster-container">
          <img
            :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
            :alt="movie.title"
            class="poster"
          />
          <div class="rating">
            <span class="star">⭐</span>
            <span>{{ movie.vote_average.toFixed(1) }}</span>
          </div>
        </div>
        <div class="movie-info">
          <h3 class="movie-title">{{ movie.title }}</h3>
          <div class="movie-meta">
            <span class="meta-item">
              <span class="calendar-icon">📅</span>
              {{ formatDate(movie.release_date) }}
            </span>
          </div>
          <div class="genre-tags">
            <span
              v-for="id in movie.genre_ids.slice(0, 2)"
              :key="id"
              class="genre-tag"
            >
              {{ getGenreName(id) }}
            </span>
          </div>
        </div>
      </div>
    </div>

    <div v-if="selectedMovie" class="modal" @click="closeModal">
      <div class="modal-content" @click.stop>
        <button class="close-button" @click="closeModal">✕</button>
        <div class="modal-grid">
          <img
            :src="`https://image.tmdb.org/t/p/w500${selectedMovie.poster_path}`"
            :alt="selectedMovie.title"
            class="modal-poster"
          />
          <div class="modal-details">
            <h2 class="modal-title">{{ selectedMovie.title }}</h2>
            <p v-if="selectedMovie.tagline" class="tagline">
              "{{ selectedMovie.tagline }}"
            </p>
            <div class="modal-meta">
              <span class="modal-meta-item">
                ⭐ {{ selectedMovie.vote_average?.toFixed(1) }} / 10
              </span>
              <span class="modal-meta-item">
                📅 {{ formatDate(selectedMovie.release_date) }}
              </span>
              <span class="modal-meta-item">
                📊 {{ selectedMovie.vote_count }} votos
              </span>
            </div>
            <p class="overview">{{ selectedMovie.overview }}</p>
            <div v-if="selectedMovie.genres" class="modal-genres">
              <span
                v-for="genre in selectedMovie.genres"
                :key="genre.id"
                class="modal-genre-tag"
              >
                {{ genre.name }}
              </span>
            </div>
            <p v-if="selectedMovie.budget > 0" class="budget-info">
              <strong>Orçamento:</strong> ${{ selectedMovie.budget.toLocaleString() }}
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #0a0a0a 0%, #1a1a1a 50%, #2a2a2a 100%);
  color: #e0e0e0;
}

.main-nav {
  height: 3rem;
  display: flex;
  background: linear-gradient(135deg, #000000 0%, #1a1a1a 100%);
  color: #fff;
  font-size: 1.2rem;
  justify-content: center;
  align-items: center;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  position: sticky;
  top: 0;
  z-index: 100;
  column-gap: 2rem;
}

.main-nav a {
  text-decoration: none;
  color: #fff;
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  transition: all 0.3s ease;
}

.main-nav a:hover {
  background-color: #2a2a2a;
}

.main-nav a.router-link-active {
  background: linear-gradient(135deg, #ffffff 0%, #e0e0e0 100%);
  color: #000;
  font-weight: 600;
}

.header {
  background: linear-gradient(135deg, #000000 0%, #1a1a1a 100%);
  padding: 2rem;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5);
  position: sticky;
  top: 0;
  z-index: 100;
}

.header-content {
  max-width: 1400px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1.5rem;
}

.logo {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.logo-title {
  font-size: 1.8rem;
  font-weight: 700;
  margin: 0;
  background: linear-gradient(135deg, #ffffff 0%, #cccccc 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.logo-subtitle {
  font-size: 0.9rem;
  color: #999;
  margin: 0;
}

.search-box {
  position: relative;
  width: 300px;
}

.search-icon {
  position: absolute;
  left: 1rem;
  top: 50%;
  transform: translateY(-50%);
  font-size: 1.2rem;
}

.search-input {
  width: 100%;
  padding: 0.8rem 1rem 0.8rem 3rem;
  border-radius: 25px;
  border: 2px solid #333;
  background: #0a0a0a;
  color: #e0e0e0;
  font-size: 1rem;
  outline: none;
  transition: all 0.3s ease;
}

.search-input:focus {
  border-color: #ffffff;
  box-shadow: 0 0 20px rgba(255, 255, 255, 0.2);
}

.filters-section {
  max-width: 1400px;
  margin: 2rem auto;
  padding: 0 2rem;
}

.filter-group {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: #999;
}

.filter-icon {
  font-size: 1.2rem;
}

.filter-label {
  font-size: 0.9rem;
  font-weight: 600;
}

.select {
  padding: 0.5rem 1rem;
  border-radius: 8px;
  border: 2px solid #333;
  background: #0a0a0a;
  color: #e0e0e0;
  font-size: 0.9rem;
  cursor: pointer;
  outline: none;
  transition: all 0.3s ease;
}

.select:hover {
  border-color: #ffffff;
}

.genres-container {
  max-width: 1400px;
  margin: 0 auto 2rem;
  padding: 0 2rem;
  display: flex;
  flex-wrap: wrap;
  gap: 0.8rem;
}

.genre-pill {
  padding: 0.6rem 1.2rem;
  border-radius: 20px;
  border: 2px solid #333;
  background: #1a1a1a;
  color: #e0e0e0;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 500;
  transition: all 0.3s ease;
  outline: none;
}

.genre-pill:hover:not(.active) {
  background: #2a2a2a;
  transform: translateY(-2px);
}

.genre-pill.active {
  background: linear-gradient(135deg, #ffffff 0%, #e0e0e0 100%);
  border-color: #ffffff;
  color: #000;
  font-weight: 700;
  transform: scale(1.05);
}

.movies-grid {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 2rem 3rem;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 2rem;
}

.movie-card {
  background: linear-gradient(135deg, #1a1a1a 0%, #0a0a0a 100%);
  border-radius: 16px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.5);
  border: 1px solid #222;
}

.movie-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 35px rgba(255, 255, 255, 0.2);
  border-color: #444;
}

.poster-container {
  position: relative;
  overflow: hidden;
}

.poster {
  width: 100%;
  height: 330px;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.movie-card:hover .poster {
  transform: scale(1.08);
}

.rating {
  position: absolute;
  top: 0.8rem;
  right: 0.8rem;
  background: rgba(0, 0, 0, 0.9);
  padding: 0.4rem 0.8rem;
  border-radius: 20px;
  display: flex;
  align-items: center;
  gap: 0.3rem;
  font-size: 0.9rem;
  font-weight: 700;
  backdrop-filter: blur(10px);
  border: 1px solid #333;
}

.star {
  font-size: 1rem;
}

.movie-info {
  padding: 1rem;
}

.movie-title {
  font-size: 1rem;
  font-weight: 700;
  margin: 0 0 0.5rem 0;
  line-height: 1.3;
  height: 2.6rem;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

.movie-meta {
  display: flex;
  gap: 1rem;
  margin-bottom: 0.8rem;
}

.meta-item {
  display: flex;
  align-items: center;
  gap: 0.3rem;
  font-size: 0.8rem;
  color: #999;
}

.calendar-icon {
  font-size: 0.9rem;
}

.genre-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
}

.genre-tag {
  padding: 0.3rem 0.7rem;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  font-size: 0.75rem;
  font-weight: 600;
  color: #e0e0e0;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.95);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 2rem;
  backdrop-filter: blur(5px);
}

.modal-content {
  background: linear-gradient(135deg, #1a1a1a 0%, #0a0a0a 100%);
  border-radius: 20px;
  max-width: 900px;
  width: 100%;
  max-height: 90vh;
  overflow: auto;
  position: relative;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.8);
  border: 1px solid #333;
}

.close-button {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid #ffffff;
  color: #fff;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  font-size: 1.5rem;
  cursor: pointer;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
}

.close-button:hover {
  background: #ffffff;
  color: #000;
  transform: rotate(90deg);
}

.modal-grid {
  display: grid;
  grid-template-columns: 300px 1fr;
  gap: 2rem;
  padding: 2rem;
}

.modal-poster {
  width: 100%;
  border-radius: 12px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.6);
}

.modal-details {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.modal-title {
  font-size: 2rem;
  font-weight: 700;
  margin: 0;
  background: linear-gradient(135deg, #ffffff 0%, #cccccc 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.tagline {
  font-style: italic;
  color: #999;
  margin: 0;
}

.modal-meta {
  display: flex;
  gap: 1.5rem;
  flex-wrap: wrap;
}

.modal-meta-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.95rem;
}

.overview {
  line-height: 1.8;
  color: #ccc;
}

.modal-genres {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.modal-genre-tag {
  padding: 0.5rem 1rem;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid rgba(255, 255, 255, 0.2);
  font-size: 0.85rem;
  font-weight: 600;
  color: #e0e0e0;
}

.budget-info {
  color: #999;
  font-size: 0.9rem;
}

@media (max-width: 768px) {
  .modal-grid {
    grid-template-columns: 1fr;
  }

  .header-content {
    flex-direction: column;
  }

  .search-box {
    width: 100%;
  }
}
</style>
