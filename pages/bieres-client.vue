<script setup>
    import { ref, onMounted } from 'vue'
    const posts = ref([])
    const getPosts = async () => {
    posts.value = await $fetch('https://api.sampleapis.com/beers/ale')
    }
    onMounted(getPosts)
</script>

<template>
<div>
    <main>
      <h1>Page bière (côté client)</h1>
      <div class="page-content">
        <p>Liste des meilleures bières IPA</p>
        <div v-if="posts.length === 0">Chargement...</div>
        <div v-else class="beers-container">
          <div v-for="post in posts" :key="post.id" class="beer-card">
            <span>{{ post.name }}</span>
            <img v-if="post.image" :src="post.image" :alt="post.name" />
            <span class="price">{{ post.price }}</span>
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

<style>

main {
  display: flex;
  flex-direction: column;
}

.beers-container {
  display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 20px;
}

.beer-card{
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
    padding: 10px;

    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}
</style>