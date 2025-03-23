<script setup>
import { ref, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter(); 
const beerType = ref(route.params.type);
const posts = ref([]);
const maxPrice = ref(route.query.pricemax || '');
const totalItems = ref(0);
const pending = ref(false);
const error = ref(false);

//favoris 
const favorites = ref([]);
const favoritesCount = ref(0);

// Pagination
const currentPage = ref(parseInt(route.query.page) || 1);
const itemsPerPage = ref(parseInt(route.query.limit) || 15);
const totalPages = ref(1);

const fetchServerPaginatedData = async () => {
  pending.value = true;
  error.value = false;

  try {
    const page = currentPage.value;
    const limit = itemsPerPage.value;
    const pricemax = maxPrice.value;

    const { data: allBeers } = await useFetch(`https://api.sampleapis.com/beers/${beerType.value}`);
    let filteredBeers = allBeers.value;

    if (pricemax) {
      filteredBeers = filteredBeers.filter(beer => {
        const price = convertPrice(beer.price);
        return price <= parseFloat(pricemax); 
      });
    }

    totalItems.value = filteredBeers.length;
    totalPages.value = Math.ceil(totalItems.value / limit);

    const startIndex = (page - 1) * limit;
    const endIndex = startIndex + limit;
    posts.value = filteredBeers.slice(startIndex, endIndex);
  } catch (err) {
    console.error('Erreur lors de la récupération des bières:', err);
    error.value = true;
    posts.value = [];
    totalItems.value = 0;
    totalPages.value = 1;
  } finally {
    pending.value = false;
  }
};

// on récupére les favoris du local storage
const loadFavorites = () => {
  try {
    const storedFavorites = localStorage.getItem('beer-favorites');
    if (storedFavorites) {
      favorites.value = JSON.parse(storedFavorites);
      favoritesCount.value = favorites.value.length;
    }
  } catch (error) {
    console.error('Erreur lors du chargement des favoris:', error);
    favorites.value = [];
  }
};

//on sauvegarde les favoris dans le local storage
const saveFavorites = () => {
  try {
    localStorage.setItem('beer-favorites', JSON.stringify(favorites.value));
    favoritesCount.value = favorites.value.length;
  } catch (error) {
    console.error('Erreur lors de la sauvegarde des favoris:', error);
  }
};

//on vériie si la bière est favorite
const isFavorite = (beer) => {
  return favorites.value.some(fav =>
    fav.id === beer.id &&
    fav.type === beerType.value
  );
};

// on ajoute ou retire une bière des favoris
const toggleFavorite = (beer) => {
  const beerWithType = {
    ...beer,
    type: beerType.value
  };

  const index = favorites.value.findIndex(fav =>
    fav.id === beer.id &&
    fav.type === beerType.value
  );

  if (index !== -1) {
    favorites.value.splice(index, 1);
  } else {
    favorites.value.push(beerWithType);
  }

  saveFavorites();
};


// por convertir le prix en nombre
const convertPrice = (priceStr) => {
  if (!priceStr) return 0;
  const priceString = String(priceStr);
  return parseFloat(priceString.replace('$', ''));
};

const updateQueryParams = () => {
  router.push({
    query: {
      pricemax: maxPrice.value || undefined,
      page: currentPage.value > 1 ? currentPage.value : undefined,
      limit: itemsPerPage.value !== 15 ? itemsPerPage.value : undefined,
    }
  });
};

const filterByPrice = () => {
  currentPage.value = 1;
  updateQueryParams();
};

//gestion de la pagination
const changePage = (newPage) => {
  if (newPage >= 1 && newPage <= totalPages.value) {
    currentPage.value = newPage;
    updateQueryParams();
  }
};

const changeItemsPerPage = () => {
  currentPage.value = 1;
  updateQueryParams();
};

watch(() => route.query, (newQuery) => {
  if (newQuery.page) {
    currentPage.value = parseInt(newQuery.page);
  }
  if (newQuery.limit) {
    itemsPerPage.value = parseInt(newQuery.limit);
  }
  if (newQuery.pricemax !== undefined) {
    maxPrice.value = newQuery.pricemax;
  }
  fetchServerPaginatedData();
}, { deep: true });

onMounted(() => {
  loadFavorites();
});

await fetchServerPaginatedData();
</script>

<template>
<div>
    <main>
      <h1>Page bière (serveur)</h1>
      <div class="page-content">
        <div class="filter">
            <input 
              type="text" 
              placeholder="Rechercher par prix" 
              v-model="maxPrice"
              @keyup.enter="filterByPrice"
            />
            <button @click="filterByPrice">Rechercher</button>

            <NuxtLink to="/favoris" class="fav-link">
              Voir mes bières favorites ({{ favoritesCount }})
            </NuxtLink>
        </div>
        <div v-if="pending">Chargement...</div>
        <div v-else-if="error">Erreur lors du chargement des données</div>
        <div v-else-if="!posts || posts.length === 0">Aucune bière trouvée</div>
        <div v-else class="beers-container">
          <div v-for="post in posts" :key="post.id" class="beer-card">
            <span>{{ post.name }}</span>
            <button 
              @click="toggleFavorite(post)" 
              class="fav-button"
              :class="{ 'is-favorite': isFavorite(post) }"
              :title="isFavorite(post) ? 'Retirer des favoris' : 'Ajouter aux favoris'"
            >
            <span>
              {{ isFavorite(post) ? '★' : '☆' }}
            </span>
            </button>
            <div class="beer-details">
              <span class="price">{{ post.price }}</span>
              <NuxtLink :to="'/bieres-serveur/' + beerType + '/' + post.id">Voir les détails</NuxtLink>
            </div>
          </div>
        </div>
      </div>

      <div class="pagination-container">
        <div class="pagination">
          <button 
            :disabled="currentPage <= 1" 
            @click="changePage(currentPage - 1)"
            class="pagination-btn"
          >
           <
          </button>
          
          <span class="page-info">
            Page {{ currentPage }} sur {{ totalPages }}
          </span>
          
          <button 
            :disabled="currentPage >= totalPages" 
            @click="changePage(currentPage + 1)"
            class="pagination-btn"
          >
            >
          </button>
        </div>
        
        <div class="items-per-page">
          <label for="itemsPerPage">Nombre d'éléments par page : </label>
          <select id="itemsPerPage" v-model="itemsPerPage" @change="changeItemsPerPage">
            <option value="15">15</option>
            <option value="30">30</option>
            <option value="45">45</option>
          </select>
        </div>
      </div>
    </main>
</div>
</template>