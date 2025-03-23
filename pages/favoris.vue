<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const favorites = ref([]);

const loadFavorites = () => {
  try {
    const storedFavorites = localStorage.getItem('beer-favorites');
    if (storedFavorites) {
      favorites.value = JSON.parse(storedFavorites);
    }
  } catch (error) {
    console.error('Erreur lors du chargement des favoris:', error);
    favorites.value = [];
  }
};

const saveFavorites = () => {
  try {
    localStorage.setItem('beer-favorites', JSON.stringify(favorites.value));
  } catch (error) {
    console.error('Erreur lors de la sauvegarde des favoris:', error);
  }
};

const removeFavorite = (beer) => {
  const index = favorites.value.findIndex(fav => 
    fav.id === beer.id && 
    fav.type === beer.type
  );
  
  if (index !== -1) {
    favorites.value.splice(index, 1);
    saveFavorites();
  }
};

const clearAllFavorites = () => {
  if (confirm('Êtes-vous sûr de vouloir supprimer tous vos favoris ?')) {
    favorites.value = [];
    saveFavorites();
  }
};

//synchro des favoris entre les onglets
const handleStorageChange = (event) => {
  if (event.key === 'beer-favorites') {
    loadFavorites();
  }
};

onMounted(() => {
  loadFavorites();
  window.addEventListener('storage', handleStorageChange);
});

onUnmounted(() => {
  window.removeEventListener('storage', handleStorageChange);
});
</script>

<template>
  <div>
    <main>
      <div class="back-link">
        <NuxtLink to="/bieres-serveur/ale">← Retour</NuxtLink>
      </div>
      <h1>Mes bières favorites</h1>
      
      <div v-if="favorites.length === 0" class="empty-state">
        <p>Vous n'avez pas encore ajouté de bières à vos favoris.</p>
        <NuxtLink to="/" class="home-link">Découvrir des bières</NuxtLink>
      </div>
      
      <div v-else>
        <div class="beers-container">
          <div v-for="beer in favorites" :key="`${beer.type}-${beer.id}`" class="beer-card">
              <span>{{ beer.name }}</span>
              <button 
                @click="removeFavorite(beer)" 
                class="fav-button is-favorite"
              >
              <span>
                ★
              </span>
              </button>
            <div class="beer-details">
              <span class="price">{{ beer.price }}</span>
              <NuxtLink :to="'/bieres-serveur/' + beer.type + '/' + beer.id">Voir les détails</NuxtLink>
            </div>
          </div>
        </div>
        
        <div class="clear-favorites">
          <button @click="clearAllFavorites" class="clear-btn">
            Supprimer tous mes favoris
          </button>
        </div>
      </div>
    </main>
  </div>
</template>