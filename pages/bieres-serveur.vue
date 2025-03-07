<script setup>
const { data: posts } = await useFetch('https://api.sampleapis.com/beers/ale')
</script>

<template>
<div>
    <main>
      <h1>Page bière (côté Serveur)</h1>
      <div class="page-content">
        <p>Liste des meilleures bières IPA</p>
        <div v-if="posts.length === 0">Chargement...</div>
        <div v-else class="beers-container">
          <div v-for="post in posts" :key="post.id" class="beer-card">
            <span>{{ post.name }}</span>
            <img :src="post.image" :alt="post.name" />
            <span class ="price">{{ post.price }}</span>
          </div>
        </div>
      </div>
      
    </main>
    
   <footer>
      <p>TP2 Bières</p>
      <p>Nicka Ratovobodo</p>
    </footer>
</div>
</template>