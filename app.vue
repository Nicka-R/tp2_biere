<script setup lang="ts">
import { NuxtPage } from '#components';

</script>

<template>
  <header>    
    <NuxtLink href="/bieres-serveur">Serveur</NuxtLink>
    <NuxtLink href="/bieres-client">Client</NuxtLink>
  </header>
  <div>
    <NuxtRouteAnnouncer />
    <NuxtPage />
  </div>
</template>

<style scoped>
header {
  background-color: rgb(252, 206, 56);
  padding: 10px;
  display: flex;
  justify-content: space-around;
  gap: 20px;

  a{
    text-decoration: none;
    color: black;

    &:hover{
      font-weight: bold;
      transform: scale(1.1);
      transition: transform 0.5s;
    }
  }
}

</style>