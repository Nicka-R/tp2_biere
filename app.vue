<script setup lang="ts">
import { NuxtPage } from '#components';

</script>

<template>
  <header>    
    <NuxtLink href="/">Accueil (choix du type de bière)</NuxtLink>
  </header>
  <div>
    <NuxtRouteAnnouncer />
    <NuxtPage />
  </div>
</template>