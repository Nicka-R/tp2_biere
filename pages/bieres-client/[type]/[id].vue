<script setup lang="ts">
import { useRoute } from 'vue-router'
import { ref, onMounted } from 'vue'

const route = useRoute()
const id = route.params.id
const post = ref(null)

const beerType = ref(route.params.type || 'ale');

const getPost = async () => {
  post.value = await $fetch(`https://api.sampleapis.com/beers/${beerType.value}/${id}`)
};
onMounted(getPost)

</script>

<template>
  <main>
    <NuxtLink :to="'/bieres-client/' + beerType" class="back-link">← Retour à la liste des {{ beerType }}</NuxtLink>
    <div class="beer-details-container" v-if="post">
       <h1>{{ post.name }}</h1>
      <div class="page-details">
        <img v-if="post.image" :src="post.image" :alt="post.name" />
        <span class="price">{{ post.price }}</span>
        <div class="beer-details">
          <div class="rating">
              <span>{{ post.rating.average.toFixed(2) }}/5 </span>
              <span>({{ post.rating.reviews }} avis)</span>
          </div>
          <button>Ajouter aux favoris</button>
        </div>
      </div>
    </div>
  </main>

</template>