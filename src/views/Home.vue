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
    <!-- Header -->
    <header class="header">
      <div class="header-content">
        <div class="logo">
          <span class="logo-icon">🎬</span>
          <div>
            <h1 class="logo-title">Filmes Medianos</h1>
            <p class="logo-subtitle">A beleza do meio-termo (5-7 ⭐)</p>
          </div>
        </div>
        <div class="search-box">
          <span class="search-icon">🔍</span>
          <input
            type="text"
            placeholder="Buscar filmes..."
            v-model="searchTerm"
            class="search-input"
          />
        </div>
      </div>
    </header>

    <!-- Filters -->
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

    <!-- Genre Pills -->
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

    <!-- Loading -->
    <loading v-model:active="isLoading" is-full-page />

    <!-- Movies Grid -->
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

    <!-- Movie Details Modal -->
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
