# Thème clair / sombre

Une recette SvelteKit + Svelte 5 + Tailwind v4. Tout le code à copier est ici : pas de store, pas de rune, pas de page de démonstration à récupérer.

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

## 1. Appliquer le thème avant le rendu — src/app.html

Ajoute le script ci-dessous dans le head, **avant %sveltekit.head%**. Voici le fichier minimal complet pour situer le script ; garde tes métadonnées et autres personnalisations si le fichier existe déjà.

```html
<!doctype html>
<html lang="fr">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <script nonce="%sveltekit.nonce%">
      (() => {
        const root = document.documentElement;
        const darkModeQuery = matchMedia('(prefers-color-scheme: dark)');

        function readStoredTheme() {
          try {
            const theme = localStorage.getItem('theme');
            return theme === 'dark' || theme === 'light' ? theme : null;
          } catch {
            return null;
          }
        }

        function applyTheme() {
          const theme = readStoredTheme() ?? (darkModeQuery.matches ? 'dark' : 'light');
          root.classList.toggle('dark', theme === 'dark');
        }

        applyTheme();
        darkModeQuery.addEventListener('change', applyTheme);
        window.addEventListener('storage', applyTheme);
      })();
    </script>
    %sveltekit.head%
  </head>
  <body data-sveltekit-preload-data="hover">
    <div style="display: contents">%sveltekit.body%</div>
  </body>
</html>
```

## 2. Activer les variantes dark — src/app.css

Ajoute ces règles à ton CSS principal. Ne répète pas l’import Tailwind s’il existe déjà.

```css
@import 'tailwindcss';

@custom-variant dark (&:where(.dark, .dark *));

@layer base {
  html { color-scheme: light; }
  html.dark { color-scheme: dark; }
}
```

## 3. Créer le bouton — src/lib/components/ThemeToggle.svelte

Copie ce composant complet :

```svelte
<script>
  function toggleTheme() {
    const root = document.documentElement;
    const dark = !root.classList.contains('dark');
    root.classList.toggle('dark', dark);

    try {
      localStorage.setItem('theme', dark ? 'dark' : 'light');
    } catch {
      // Le bouton fonctionne aussi lorsque le stockage est indisponible.
    }
  }
</script>

<button type="button" onclick={toggleTheme} class="rounded-lg border px-4 py-2">
  <span class="dark:hidden">Passer au sombre</span>
  <span class="hidden dark:inline">Passer au clair</span>
</button>
```

## 4. Brancher le bouton — src/routes/+layout.svelte

Exemple minimal d’intégration. Dans un layout existant, ajoute les imports et place le bouton où tu le souhaites ; garde ton contenu et ton rendu children actuels.

```svelte
<script>
  import '../app.css';
  import ThemeToggle from '$lib/components/ThemeToggle.svelte';
  let { children } = $props();
</script>

<ThemeToggle />
{@render children()}
```

## 5. Choisir les couleurs dans tes composants

La classe dark ne choisit pas les couleurs de ton interface à ta place. Ajoute les variantes sur les éléments concernés, par exemple :

```svelte
<div class="bg-white text-stone-900 dark:bg-stone-950 dark:text-stone-100">
  Mon contenu
</div>
```

Le reset, la police, le curseur et le focus clavier sont indépendants du thème : voir [Styles de base](base-styles.md). Le bouton conserve le focus natif ; tu peux le personnaliser globalement ou avec les utilitaires de cette fiche.

## Comprendre

Le script est exécuté par le navigateur pendant la lecture du HTML, avant le contenu et l’hydratation Svelte. Il applique .dark sur html : le thème n’attend pas onMount.

readStoredTheme lit light ou dark dans localStorage. Si la clé est absente, invalide ou inaccessible, darkModeQuery.matches fournit la préférence système. Le changement système relance applyTheme ; un choix enregistré reste prioritaire.

Le bouton lit la classe réelle de html, la bascule, puis sauvegarde le choix. Les libellés sont affichés par CSS (dark:hidden), sans état JavaScript supplémentaire. color-scheme adapte les champs natifs et les barres de défilement.

L’événement storage relance applyTheme dans les autres onglets de la même origine. Le bouton applique directement le changement dans son propre onglet, où cet événement ne se déclenche pas. Effacer la clé ou le stockage est aussi pris en compte.

## Limites assumées

Le bouton reste utilisable si le stockage est bloqué, mais le choix ne persiste pas ; un changement système ultérieur peut alors le remplacer. Sans JavaScript, la page reste claire. Une CSP doit autoriser le script via la configuration SvelteKit ; son nonce est prévu dans le template.

Pas de troisième bouton système : pour revenir au comportement automatique, supprimer la clé theme et recharger. Les écouteurs vivent pendant la durée du document, sans être recréés à chaque navigation.

## Vérifier

Tester sans clé avec système clair/sombre, enregistrer un choix et recharger, changer le système après un choix explicite, puis essayer deux onglets. Tester aussi une valeur invalide et un stockage inaccessible. Ces vérifications se font dans ton projet cible.

Références : https://tailwindcss.com/docs/dark-mode et https://svelte.dev/docs/kit/configuration#csp.
