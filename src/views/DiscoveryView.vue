<script setup>
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import Loading from 'vue-loading-overlay';
import api from '@/plugins/axios';

const router = useRouter();

const randomMovie = ref(null);
const trendingMovies = ref([]);
const recentReleases = ref([]);
const similarMovies = ref([]);
const selectedMovieForSimilar = ref(null);
const isLoading = ref(false);
const isLoadingSimilar = ref(false);

const getRandomMovie = async () => {
  isLoading.value = true;
  try {
    const randomPage = Math.floor(Math.random() * 10) + 1;
    const response = await api.get('discover/movie', {
      params: {
        language: 'pt-BR',
        sort_by: 'popularity.desc',
        'vote_average.gte': 5,
        'vote_average.lte': 7,
        'vote_count.gte': 100,
        page: randomPage,
      },
    });

    const movies = response.data.results || [];
    if (movies.length > 0) {
      const randomIndex = Math.floor(Math.random() * movies.length);
      randomMovie.value = movies[randomIndex];
    }
  } catch (error) {
    console.error('Erro ao buscar filme aleatório:', error);
  } finally {
    isLoading.value = false;
  }
};

const fetchTrending = async () => {
  try {
    const response = await api.get('trending/movie/week', {
      params: {
        language: 'pt-BR',
      },
    });

    trendingMovies.value = (response.data.results || [])
      .filter(movie => movie.vote_average >= 5 && movie.vote_average <= 7)
      .slice(0, 8);
  } catch (error) {
    console.error('Erro ao buscar trending:', error);
  }
};

const fetchRecentReleases = async () => {
  try {
    const currentDate = new Date();
    const threeMonthsAgo = new Date(currentDate.setMonth(currentDate.getMonth() - 3));
    const formattedDate = threeMonthsAgo.toISOString().split('T')[0];

    const response = await api.get('discover/movie', {
      params: {
        language: 'pt-BR',
        sort_by: 'release_date.desc',
        'release_date.gte': formattedDate,
        'vote_average.gte': 5,
        'vote_average.lte': 7,
        'vote_count.gte': 50,
      },
    });

    recentReleases.value = (response.data.results || []).slice(0, 8);
  } catch (error) {
    console.error('Erro ao buscar lançamentos:', error);
  }
};

const searchSimilarMovies = async () => {
  if (!selectedMovieForSimilar.value) return;

  isLoadingSimilar.value = true;
  try {
    const response = await api.get(`movie/${selectedMovieForSimilar.value}/similar`, {
      params: {
        language: 'pt-BR',
      },
    });

    similarMovies.value = (response.data.results || [])
      .filter(movie => movie.vote_average >= 5 && movie.vote_average <= 7)
      .slice(0, 8);
  } catch (error) {
    console.error('Erro ao buscar similares:', error);
  } finally {
    isLoadingSimilar.value = false;
  }
};

const formatDate = (date) => {
  if (!date) return 'Data desconhecida';
  return new Date(date).toLocaleDateString('pt-BR');
};

const openMovieDetails = (movieId) => {
  router.push({ name: 'MovieDetails', params: { movieId } });
};

onMounted(async () => {
  isLoading.value = true;
  await Promise.all([
    getRandomMovie(),
    fetchTrending(),
    fetchRecentReleases(),
  ]);
  isLoading.value = false;
});
</script>

<template>
  <div class="discovery-container">
    <div class="header-section">
      <h1 class="page-title">
        <span class="title-icon">🎲</span>
        Descoberta
      </h1>
      <p class="subtitle">Explore novos filmes na medida perfeita</p>
    </div>

    <loading v-model:active="isLoading" is-full-page />

    <section class="random-section">
      <div class="section-header">
        <h2>🎲 Filme Aleatório</h2>
        <button @click="getRandomMovie" class="shuffle-btn">
           Novo Filme
        </button>
      </div>

      <div v-if="randomMovie" class="random-movie-card">
        <img
          :src="`https://image.tmdb.org/t/p/w500${randomMovie.poster_path}`"
          :alt="randomMovie.title"
          class="random-poster"
          @click="openMovieDetails(randomMovie.id)"
        />
        <div class="random-details">
          <h3 class="random-title">{{ randomMovie.title }}</h3>
          <div class="random-meta">
            <span class="rating">⭐ {{ randomMovie.vote_average.toFixed(1) }}</span>
            <span class="date">📅 {{ formatDate(randomMovie.release_date) }}</span>
          </div>
          <p class="random-overview">{{ randomMovie.overview }}</p>
          <button @click="openMovieDetails(randomMovie.id)" class="details-btn">
            Ver Detalhes →
          </button>
        </div>
      </div>
    </section>

    <section class="movies-section">
      <div class="section-header">
        <h2>Em Alta esta Semana</h2>
      </div>

      <div class="movies-grid">
        <div
          v-for="movie in trendingMovies"
          :key="movie.id"
          class="movie-card"
          @click="openMovieDetails(movie.id)"
        >
          <div class="poster-container">
            <img
              :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
              :alt="movie.title"
              class="poster"
            />
            <div class="rating-badge">
              <span class="star">⭐</span>
              <span>{{ movie.vote_average.toFixed(1) }}</span>
            </div>
          </div>
          <div class="movie-info">
            <h3 class="movie-title">{{ movie.title }}</h3>
            <p class="movie-date">{{ formatDate(movie.release_date) }}</p>
          </div>
        </div>
      </div>
    </section>

    <section class="movies-section">
      <div class="section-header">
        <h2>🆕 Lançamentos Recentes</h2>
      </div>

      <div class="movies-grid">
        <div
          v-for="movie in recentReleases"
          :key="movie.id"
          class="movie-card"
          @click="openMovieDetails(movie.id)"
        >
          <div class="poster-container">
            <img
              :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
              :alt="movie.title"
              class="poster"
            />
            <div class="rating-badge">
              <span class="star">⭐</span>
              <span>{{ movie.vote_average.toFixed(1) }}</span>
            </div>
          </div>
          <div class="movie-info">
            <h3 class="movie-title">{{ movie.title }}</h3>
            <p class="movie-date">{{ formatDate(movie.release_date) }}</p>
          </div>
        </div>
      </div>
    </section>

    <!-- Filmes Similares -->
    <section class="similar-section">
      <div class="section-header">
        <h2>🎬 Encontrar Similares</h2>
      </div>

      <div class="similar-search">
        <input
          v-model.number="selectedMovieForSimilar"
          type="number"
          placeholder="Digite o ID do filme..."
          class="similar-input"
          @keyup.enter="searchSimilarMovies"
        />
        <button @click="searchSimilarMovies" class="search-btn">
          Buscar Similares
        </button>
      </div>

      <loading v-model:active="isLoadingSimilar" />

      <div v-if="similarMovies.length > 0" class="movies-grid">
        <div
          v-for="movie in similarMovies"
          :key="movie.id"
          class="movie-card"
          @click="openMovieDetails(movie.id)"
        >
          <div class="poster-container">
            <img
              :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
              :alt="movie.title"
              class="poster"
            />
            <div class="rating-badge">
              <span class="star">⭐</span>
              <span>{{ movie.vote_average.toFixed(1) }}</span>
            </div>
          </div>
          <div class="movie-info">
            <h3 class="movie-title">{{ movie.title }}</h3>
            <p class="movie-date">{{ formatDate(movie.release_date) }}</p>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<style scoped>
.discovery-container {
  min-height: 100vh;
  padding: 2rem;
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.header-section {
  text-align: center;
  margin-bottom: 3rem;
}

.page-title {
  font-size: 2.5rem;
  font-weight: 700;
  margin-bottom: 0.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  background: linear-gradient(135deg, #ffffff 0%, #cccccc 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.title-icon {
  font-size: 3rem;
  filter: drop-shadow(0 0 20px rgba(255, 255, 255, 0.3));
}

.subtitle {
  color: #999;
  font-size: 1.1rem;
}

.random-section {
  max-width: 1200px;
  margin: 0 auto 4rem;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.section-header h2 {
  font-size: 1.8rem;
  font-weight: 700;
  color: #fff;
}

.shuffle-btn {
  padding: 0.8rem 1.5rem;
  background: linear-gradient(135deg, #ffffff 0%, #e0e0e0 100%);
  color: #000;
  border: none;
  border-radius: 25px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 1rem;
}

.shuffle-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(255, 255, 255, 0.3);
}

.random-movie-card {
  display: grid;
  grid-template-columns: 300px 1fr;
  gap: 2rem;
  background: linear-gradient(135deg, #1a1a1a 0%, #0a0a0a 100%);
  border-radius: 20px;
  padding: 2rem;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5);
  border: 1px solid #333;
}

.random-poster {
  width: 100%;
  border-radius: 12px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.6);
  cursor: pointer;
  transition: transform 0.3s ease;
}

.random-poster:hover {
  transform: scale(1.05);
}

.random-details {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  justify-content: center;
}

.random-title {
  font-size: 2rem;
  font-weight: 700;
  margin: 0;
  background: linear-gradient(135deg, #ffffff 0%, #cccccc 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.random-meta {
  display: flex;
  gap: 2rem;
  font-size: 1.1rem;
}

.rating {
  color: #ffd700;
  font-weight: 700;
}

.date {
  color: #999;
}

.random-overview {
  line-height: 1.8;
  color: #ccc;
  font-size: 1rem;
}

.details-btn {
  padding: 1rem 2rem;
  background: transparent;
  color: #fff;
  border: 2px solid #fff;
  border-radius: 25px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 1rem;
  width: fit-content;
}

.details-btn:hover {
  background: #fff;
  color: #000;
  transform: translateX(5px);
}

.movies-section {
  max-width: 1400px;
  margin: 0 auto 4rem;
}

.movies-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1.5rem;
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
  height: 300px;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.movie-card:hover .poster {
  transform: scale(1.08);
}

.rating-badge {
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
  color: #fff;
}

.movie-date {
  font-size: 0.85rem;
  color: #999;
  margin: 0;
}

.similar-section {
  max-width: 1400px;
  margin: 0 auto 4rem;
}

.similar-search {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
  max-width: 600px;
}

.similar-input {
  flex: 1;
  padding: 1rem 1.5rem;
  border-radius: 25px;
  border: 2px solid #333;
  background: #0a0a0a;
  color: #e0e0e0;
  font-size: 1rem;
  outline: none;
  transition: all 0.3s ease;
}

.similar-input:focus {
  border-color: #ffffff;
  box-shadow: 0 0 20px rgba(255, 255, 255, 0.2);
}

.search-btn {
  padding: 1rem 2rem;
  background: linear-gradient(135deg, #ffffff 0%, #e0e0e0 100%);
  color: #000;
  border: none;
  border-radius: 25px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 1rem;
  white-space: nowrap;
}

.search-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(255, 255, 255, 0.3);
}

@media (max-width: 768px) {
  .random-movie-card {
    grid-template-columns: 1fr;
  }

  .similar-search {
    flex-direction: column;
  }

  .page-title {
    font-size: 2rem;
  }
}
</style>
