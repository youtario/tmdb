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
