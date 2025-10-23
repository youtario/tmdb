<script setup>
import { ref, onMounted } from 'vue';
import Loading from 'vue-loading-overlay';
import api from '@/plugins/axios';
import { useGenreStore } from '@/stores/genre';

const genreStore = useGenreStore();

const tvShows = ref([]);
const isLoading = ref(false);

const listTvShows = async (genreId) => {
  genreStore.setCurrentGenreId(genreId);
  isLoading.value = true;
  try {
    const response = await api.get('discover/tv', {
      params: {
        with_genres: genreId,
        language: 'pt-BR',
      },
    });
    tvShows.value = response.data.results;
  } catch (error) {
    console.error('Erro ao buscar programas de TV:', error);
  } finally {
    isLoading.value = false;
  }
};

const formatDate = (date) => {
  if (!date) return 'Data desconhecida';
  return new Date(date).toLocaleDateString('pt-BR');
};

onMounted(async () => {
  isLoading.value = true;
  await genreStore.getAllGenres('tv');
  isLoading.value = false;
});
</script>

<template>
  <div class="tv-container">
    <div class="header-section">
      <h1 class="page-title">
        <span class="title-icon">📺</span>
        Programas de TV
      </h1>
      <p class="subtitle">Descubra as melhores séries e programas</p>
    </div>

    <div class="genre-section">
      <ul class="genre-list">
        <li
          v-for="genre in genreStore.genres"
          :key="genre.id"
          @click="listTvShows(genre.id)"
          class="genre-item"
          :class="{ active: genre.id === genreStore.currentGenreId }"
        >
          {{ genre.name }}
        </li>
      </ul>
    </div>

    <loading v-model:active="isLoading" is-full-page />

    <div class="show-list" v-if="tvShows.length > 0">
      <div v-for="show in tvShows" :key="show.id" class="show-card">
        <div class="show-poster">
          <img
            :src="`https://image.tmdb.org/t/p/w500${show.poster_path}`"
            :alt="show.name"
          />
          <div class="poster-overlay">
            <span class="info-icon">ⓘ</span>
          </div>
        </div>
        <div class="show-details">
          <p class="show-title">{{ show.name }}</p>
          <div class="show-meta">
            <span class="show-rating">⭐ {{ show.vote_average?.toFixed(1) }}</span>
            <span class="show-release-date">{{ formatDate(show.first_air_date) }}</span>
          </div>
          <div class="show-genres">
            <span
              v-for="genre_id in show.genre_ids.slice(0, 3)"
              :key="genre_id"
              @click="listTvShows(genre_id)"
              :class="{ active: genre_id === genreStore.currentGenreId }"
            >
              {{ genreStore.getGenreName(genre_id) }}
            </span>
          </div>
        </div>
      </div>
    </div>

    <div v-else-if="!isLoading && genreStore.currentGenreId" class="empty-state">
      <p>Nenhum programa encontrado para este gênero.</p>
    </div>
  </div>
</template>

<style scoped>
.tv-container {
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
  background: linear-gradient(135deg, #fff 0%, #e0e0e0 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.title-icon {
  font-size: 3rem;
  filter: drop-shadow(0 0 20px rgba(93, 100, 36, 0.5));
}

.subtitle {
  color: #999;
  font-size: 1.1rem;
}

.genre-section {
  margin-bottom: 2rem;
}

.genre-list {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 0.8rem;
  list-style: none;
  padding: 0;
}

.genre-item {
  background: linear-gradient(135deg, #4a5020 0%, #5d6424 100%);
  border-radius: 25px;
  padding: 0.7rem 1.5rem;
  color: #fff;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid transparent;
  position: relative;
  overflow: hidden;
}

.genre-item::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.2);
  transform: translate(-50%, -50%);
  transition: width 0.6s, height 0.6s;
}

.genre-item:hover::before {
  width: 300px;
  height: 300px;
}

.genre-item:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(93, 100, 36, 0.4);
  border-color: #7d8a2e;
}

.genre-item.active {
  background: linear-gradient(135deg, #abc322 0%, #8ea00a 100%);
  color: #000;
  font-weight: 700;
  box-shadow: 0 6px 20px rgba(171, 195, 34, 0.5);
  transform: scale(1.05);
}

.show-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 2rem;
  animation: slideUp 0.6s ease;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.show-card {
  background: linear-gradient(135deg, #1a1a1a 0%, #2a2a2a 100%);
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
  transition: all 0.4s ease;
  position: relative;
}

.show-card:hover {
  transform: translateY(-10px) scale(1.02);
  box-shadow: 0 20px 50px rgba(171, 195, 34, 0.3);
}

.show-poster {
  position: relative;
  overflow: hidden;
  cursor: pointer;
  height: 360px;
}

.show-poster img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.4s ease;
}

.show-card:hover .show-poster img {
  transform: scale(1.1);
}

.poster-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(to top, rgba(0, 0, 0, 0.9) 0%, transparent 50%);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.show-card:hover .poster-overlay {
  opacity: 1;
}

.info-icon {
  font-size: 4rem;
  color: #abc322;
  filter: drop-shadow(0 0 20px rgba(171, 195, 34, 0.8));
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
}

.show-details {
  padding: 1.2rem;
}

.show-title {
  font-size: 1.1rem;
  font-weight: 700;
  line-height: 1.4;
  height: 2.8rem;
  margin-bottom: 0.8rem;
  color: #fff;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

.show-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.8rem;
  padding-bottom: 0.8rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.show-rating {
  color: #ffd700;
  font-weight: 600;
  font-size: 0.95rem;
}

.show-release-date {
  color: #999;
  font-size: 0.85rem;
}

.show-genres {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
}

.show-genres span {
  background: linear-gradient(135deg, #748708 0%, #5a6a06 100%);
  border-radius: 12px;
  padding: 0.3rem 0.8rem;
  color: #fff;
  font-size: 0.75rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid transparent;
}

.show-genres span:hover {
  background: linear-gradient(135deg, #8ea00a 0%, #748708 100%);
  box-shadow: 0 4px 15px rgba(116, 135, 8, 0.5);
  transform: translateY(-2px);
  border-color: #abc322;
}

.show-genres span.active {
  background: linear-gradient(135deg, #abc322 0%, #8ea00a 100%);
  color: #000;
  font-weight: 700;
  box-shadow: 0 4px 15px rgba(171, 195, 34, 0.5);
}

.empty-state {
  text-align: center;
  padding: 4rem 2rem;
  color: #999;
  font-size: 1.2rem;
}

@media (max-width: 768px) {
  .page-title {
    font-size: 2rem;
  }

  .show-list {
    grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
    gap: 1.5rem;
  }

  .show-poster {
    height: 240px;
  }
}
</style>
