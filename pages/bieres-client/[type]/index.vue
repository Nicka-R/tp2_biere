<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();

const beerType = ref(route.params.type || 'ale');
const posts = ref([]);
const maxPrice = ref(route.query.pricemax || '');

const currentPage = ref(parseInt(route.query.page) || 1);
const itemsPerPage = ref(parseInt(route.query.limit) || 15);

const getPosts = async () => {
  posts.value = await $fetch(`https://api.sampleapis.com/beers/${beerType.value}`);
};

const convertPrice = (priceStr) => {
  if (!priceStr) return 0;
  const priceString = String(priceStr);
  return parseFloat(priceString.replace('$', ''));
};

const filteredPosts = computed(() => {
  if (!posts.value) return [];
  if (!maxPrice.value) return posts.value;

  const maxPriceValue = parseFloat(maxPrice.value);
  if (isNaN(maxPriceValue)) return posts.value;

  return posts.value.filter(post => {
    const postPrice = convertPrice(post.price);
    return postPrice <= maxPriceValue;
  });
});

const filterByPrice = () => {
  currentPage.value = 1;
  updateQueryParams();
};

const totalPages = computed(() => {
  return Math.ceil(filteredPosts.value.length / itemsPerPage.value);
});

const paginatedPosts = computed(() => {
  const startIndex = (currentPage.value - 1) * itemsPerPage.value;
  const endIndex = startIndex + itemsPerPage.value;
  return filteredPosts.value.slice(startIndex, endIndex);
});

const updateQueryParams = () => {
  router.push({
    query: {
      pricemax: maxPrice.value || undefined,
      page: currentPage.value > 1 ? currentPage.value : undefined,
      limit: itemsPerPage.value !== 15 ? itemsPerPage.value : undefined,
    }
  });
};

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
}, { deep: true });

watch(() => route.params.type, (newType) => {
  if (newType && newType !== beerType.value) {
    beerType.value = newType;
    currentPage.value = 1;
    getPosts();
  }
}, { immediate: true });

onMounted(() => {
  getPosts();
});
</script>

<template>
<div>
    <main>
      <h1>Page bière (client)</h1>
      <div class="page-content">
        <div class="filter">
          <input 
            type="text" 
            placeholder="Rechercher par prix" 
            v-model="maxPrice"
            @keyup.enter="filterByPrice"
          />
          <button @click="filterByPrice">Rechercher</button>
        </div>
        <div v-if="filteredPosts.length === 0">Chargement...</div>
        <div v-else class="beers-container">
          <div v-for="post in paginatedPosts" :key="post.id" class="beer-card">
            <span>{{ post.name }}</span>
            <div class="beer-details">
              <span class="price">{{ post.price }}</span>
              <NuxtLink :to="'/bieres-client/' + beerType + '/' + post.id">Voir les détails</NuxtLink>
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