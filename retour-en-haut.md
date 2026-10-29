# Bouton retour en haut

Un bouton apparaît après 300 px de défilement et ramène en haut. Recette facultative, indépendante de la navigation.

## Prérequis

Dans ton projet SvelteKit : Svelte 5 récent avec prefersReducedMotion (version déclarée dans le projet d’origine : ^5.39.0) et Tailwind v4 déjà chargé. Si nécessaire, l’installation Tailwind est détaillée dans [Navigation](navigation.md#avant-de-copier).

## 1. Créer src/lib/components/BackToTop.svelte

```svelte
<script>
  import { prefersReducedMotion } from 'svelte/motion';

  let scrollY = $state(0);

  function scrollToTop() {
    window.scrollTo({
      top: 0,
      behavior: prefersReducedMotion.current ? 'instant' : 'smooth'
    });
  }
</script>

<svelte:window bind:scrollY />

{#if scrollY > 300}
  <button
    type="button"
    onclick={scrollToTop}
    class="fixed right-5 bottom-5 z-50 rounded-full border border-stone-300 bg-white px-4 py-3 text-stone-900 shadow-lg dark:border-stone-600 dark:bg-stone-800 dark:text-stone-100"
  >
    <span aria-hidden="true">↑</span> En haut
  </button>
{/if}
```

## 2. Ajouter le bouton à ta page ou à ton layout

Ajoute cet import au script existant, puis le composant après ton contenu :

```svelte
<script>
  import BackToTop from '$lib/components/BackToTop.svelte';
</script>

<BackToTop />
```

## Comprendre et adapter

scrollY suit le défilement de window. Le bouton apparaît lorsque scrollY dépasse 300 ; adapte ce seuil si besoin. scrollToTop respecte la préférence de mouvement réduit. Les classes fixed le placent en bas à droite.

Les variantes dark sont facultatives : pour piloter le sombre avec une classe sur html, utilise la [recette thème](theme.md). Sans elle, Tailwind utilise sa variante dark par défaut fondée sur le système. Pour personnaliser le focus, voir [Styles de base](base-styles.md).

## Vérifier

Dans une page assez longue, descends au-delà de 300 px et active le bouton. Vérifie aussi le clavier, le mouvement réduit et l’absence de chevauchement avec d’autres boutons fixes.
