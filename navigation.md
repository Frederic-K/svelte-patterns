# Navigation avec trait animé et menu mobile

Recette issue du Header de A K Boulangerie. Quatre liens, un trait qui se déplace sur ordinateur et un menu vertical sur mobile. Le trait suit la sélection et l’ancre de l’adresse, pas la position au défilement.

## Avant de copier

Dans ton projet SvelteKit avec Svelte 5, Tailwind v4 doit être installé. Si ce n’est pas déjà fait, exécute dans **ton projet cible** :

```sh
npm install -D tailwindcss @tailwindcss/vite
```

Dans **vite.config.js**, ajoute le plugin Tailwind en conservant les autres réglages. Configuration minimale :

```js
import { defineConfig } from 'vite';
import { sveltekit } from '@sveltejs/kit/vite';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({ plugins: [tailwindcss(), sveltekit()] });
```

Importe ensuite ton CSS principal une seule fois dans **src/routes/+layout.svelte**. Les blocs ci-dessous indiquent quoi ajouter ou créer ; conserve le contenu existant de ton projet.

Cette version utilise aussi prefersReducedMotion de Svelte 5 récent (le projet d’origine déclare Svelte ^5.39.0) et resolve de SvelteKit (projet d’origine : ^2.37.0).

## 1. Ajouter les couleurs — src/app.css

Conserve tes styles existants. Importe Tailwind une seule fois et ajoute cette palette :

```css
@import 'tailwindcss';

@theme {
	--color-bakery-cream: #f7f2ea;
	--color-bakery-sand: #ede4d6;
	--color-bakery-dark: #1e1a16;
	--color-bakery-brown: #3a2a1f;
	--color-bakery-muted: #6b5b4d;
	--color-bakery-accent: #b5562f;
	--color-bakery-accent-soft: #e2a27f;
	--color-bakery-dark-section: #2b2019;
	--color-bakery-on-dark: #d8ccbc;
	--color-bakery-line-dark: #4a3a2e;
	--color-bakery-line-soft: #c9b9a3;
}
```

## 2. Créer le conteneur — src/lib/components/Container.svelte

```svelte
<script>
	let { children } = $props();
</script>

<div class="mx-auto max-w-300">{@render children()}</div>
```

## 3. Créer la navigation — src/lib/components/Header.svelte

Copie ce composant complet :

```svelte
<script>
	import { onMount } from 'svelte';
	import { resolve } from '$app/paths';
	import { slide } from 'svelte/transition';
	import { prefersReducedMotion } from 'svelte/motion';
	import Container from '$lib/components/Container.svelte';
	let navLinks = [
		{ href: '#accueil', label: 'Accueil' },
		{ href: '#produits', label: 'Produits' },
		{ href: '#savoir-faire', label: 'Savoir-faire' },
		{ href: '#horaires', label: 'Horaires' }
	];
	let activeNavLink = $state('#accueil');
	let menuOpen = $state(false);

	// Aligne le lien sélectionné sur l’ancre de l’adresse, pas sur le défilement.
	function updateActiveLink() {
		// Exemple : /#produits donne "#produits". Sans ancre, on obtient "".
		const hash = window.location.hash;
		// some() vérifie si au moins un lien correspond à cette ancre.
		// Si oui, on la sélectionne ; sinon, on revient à Accueil.
		activeNavLink = navLinks.some((link) => link.href === hash) ? hash : '#accueil';
	}

	function closeMenu() {
		menuOpen = false;
	}

	function handleKeydown(event) {
		if (event.key === 'Escape') closeMenu();
	}

	// Au montage dans le navigateur, sélectionne le bon lien si l’on arrive
	// directement sur /#produits. window n’existe pas côté serveur.
	// On passe la fonction sans () : Svelte l’appellera au moment du montage.
	onMount(updateActiveLink);
</script>

<!-- Ensuite, hashchange actualise la sélection quand l’ancre change,
     notamment avec précédent/suivant. onkeydown permet de fermer le menu avec Échap. -->
<svelte:window onkeydown={handleKeydown} onhashchange={updateActiveLink} />

<header class="relative px-5 py-4.5 md:px-10 lg:py-7 xl:px-30">
	<Container>
		<div class="flex items-center justify-between gap-4">
			<a href={resolve('/')} class="flex items-center gap-2.5 font-serif lg:gap-3">
				<div
					class="flex size-10 items-center justify-center rounded-full border-[1.5px] border-bakery-dark text-xl leading-none tracking-tighter text-bakery-dark lg:size-11 lg:text-[22px]"
				>
					<span>A</span><span class="text-bakery-accent italic">K</span>
				</div>
				<span class="text-[21px] text-bakery-dark lg:text-[26px]">A K Boulangerie</span>
			</a>
			<nav aria-label="Navigation principale" class="relative hidden grid-cols-4 lg:grid">
				{#each navLinks as { href, label } (href)}
					<a
						{href}
						aria-current={activeNavLink === href ? "location" : undefined}
						class="z-10 px-3 py-2 text-center text-bakery-dark hover:text-bakery-accent-soft"
						onclick={() => (activeNavLink = href)}>{label}</a
					>
				{/each}
				<div
					class="absolute bottom-0 left-0 h-px w-1/4 bg-bakery-muted transition-transform duration-200 motion-reduce:transition-none"
					class:translate-x-0={activeNavLink === '#accueil'}
					class:translate-x-full={activeNavLink === '#produits'}
					class:translate-x-[200%]={activeNavLink === '#savoir-faire'}
					class:translate-x-[300%]={activeNavLink === '#horaires'}
					aria-hidden="true"
				></div>
			</nav>
			<div class="flex items-center gap-2">
				<a
					href="#nous-trouver"
					class="shrink-0 rounded-full border border-bakery-dark px-5.5 py-2.5 text-sm font-semibold text-bakery-dark hover:bg-bakery-dark hover:text-bakery-sand max-lg:hidden"
				>
					Nous trouver
				</a>
				<button
					type="button"
					class="flex size-11 items-center justify-center rounded-full border border-bakery-dark text-bakery-dark lg:hidden"
					aria-expanded={menuOpen}
					aria-controls="menu-mobile"
					aria-label={menuOpen ? 'Fermer le menu' : 'Ouvrir le menu'}
					onclick={() => (menuOpen = !menuOpen)}
				>

					<span aria-hidden="true" class="flex h-3.5 w-5 flex-col justify-between">
						<span
							class={[
								'h-0.5 rounded-full bg-current transition duration-200 motion-reduce:transition-none',
								menuOpen && 'translate-y-1.5 rotate-45'
							]}
						></span>
						<span
							class={[
								'h-0.5 rounded-full bg-current transition duration-200 motion-reduce:transition-none',
								menuOpen && 'opacity-0'
							]}
						></span>
						<span
							class={[
								'h-0.5 rounded-full bg-current transition duration-200 motion-reduce:transition-none',
								menuOpen && '-translate-y-1.5 -rotate-45'
							]}
						></span>
					</span>
				</button>
			</div>
		</div>
	</Container>

	{#if menuOpen}
		<div
			id="menu-mobile"
			class="absolute inset-x-0 top-full z-40 bg-bakery-cream px-5 pb-6 shadow-lg md:px-10 lg:hidden"
			transition:slide={{ duration: prefersReducedMotion.current ? 0 : 200 }}
		>
			<nav aria-label="Navigation mobile">
				<ul>
					{#each navLinks as { href, label } (href)}
						<li class="border-b border-bakery-line-soft/50">
							<a
								{href}
								aria-current={activeNavLink === href ? "location" : undefined}
								class="block py-3.5 text-lg font-medium text-bakery-dark hover:text-bakery-accent"
								onclick={() => { activeNavLink = href; closeMenu(); }}>{label}</a
							>
						</li>
					{/each}
				</ul>
			</nav>
			<a
				href="#nous-trouver"
				class="mt-6 block rounded-full bg-bakery-dark py-3.75 text-center text-[15px] font-semibold text-bakery-cream"
				onclick={closeMenu}
			>
				Nous trouver
			</a>
		</div>
	{/if}
</header>
```

## 4. Placer la navigation dans ta page

Dans **src/routes/+layout.svelte**, ajoute l’import du CSS principal si nécessaire :

```svelte
<script>
  import '../app.css';
  let { children } = $props();
</script>

{@render children()}
```

Dans ta page cible (par exemple **src/routes/+page.svelte**), ajoute l’import et le Header. L’enveloppe conserve les couleurs claires de cette navigation, même si le reste du site utilise un thème sombre :

```svelte
<script>
  import Header from '$lib/components/Header.svelte';
</script>

<div class="bg-bakery-cream text-bakery-dark" style="color-scheme: light">
  <Header />
</div>
```

Donne à tes vraies sections les id **accueil**, **produits**, **savoir-faire**, **horaires** et **nous-trouver**. Exemple de destination pour le lien Produits :

```html
<section id="produits">
  <h2>Nos produits</h2>
</section>
```

Pas de sections factices à recopier : utilise ton contenu. Adapte le nom, le logo et son lien resolve('/') à ta page cible. Les polices externes du projet d’origine ne sont pas nécessaires.

## Comprendre et adapter

Le desktop utilise quatre colonnes égales ; le trait mesure un quart de la barre. Les quatre classes translate-x déplacent ce trait de sa propre largeur, sans mesure du DOM. Si tu changes le nombre de liens, adapte ensemble navLinks, grid-cols-4, w-1/4 et les classes de déplacement.

menuOpen pilote le menu mobile et la transition slide. Un clic sur un lien ou Échap ferme le menu. prefersReducedMotion et motion-reduce désactivent les animations quand le système le demande. Sur mobile, la navigation est verticale, sans trait horizontal.

onMount initialise la sélection à partir de l’adresse ; hashchange suit les ancres et précédent/suivant. Une ancre inconnue sélectionne Accueil. Ce composant n’implémente pas de scrollspy.

Les styles de focus et le curseur sont un choix indépendant : voir [Styles de base](base-styles.md). Le [retour en haut](retour-en-haut.md) est lui aussi facultatif.

## Vérifier dans ton projet

Essaie les quatre liens et Nous trouver, puis une arrivée directe avec #produits et précédent/suivant. Vérifie le menu à largeur mobile, la fermeture avec Échap, la navigation clavier et la préférence système de mouvement réduit.
