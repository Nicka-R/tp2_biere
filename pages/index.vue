<template>
  <div>
    <main>
        <h1>Choisissez un type de bière</h1>
        <div class="beer-types">
            <div>
              <h2>Requête côté client</h2>
              <NuxtLink to="/bieres-client/ale">IPA</NuxtLink>
              <NuxtLink to="/bieres-client/stouts">Stouts</NuxtLink>
            </div>

            <div>
                <h2>Requête côté serveur</h2>
                <NuxtLink to="/bieres-serveur/ale">IPA</NuxtLink>
                <NuxtLink to="/bieres-serveur/stouts">Stouts</NuxtLink>
            </div>
        </div>
    </main>
  </div>
</template>
