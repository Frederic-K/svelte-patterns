# Styles de base et accessibilité

Ces styles concernent toutes les pages, quel que soit leur thème. Ils sont indépendants du mécanisme clair/sombre.

## Reset et police

Avec `@import 'tailwindcss'`, Tailwind v4 charge Preflight : les marges par défaut, dont celles du body, sont déjà supprimées. Inutile de répéter `body { margin: 0; }`.

Preflight fournit aussi une police sans empattement par défaut, selon la configuration du thème Tailwind. On peut personnaliser la police avec les variables de thème (par exemple `--font-sans`) ou utiliser `font-sans`. Tu peux utiliser cette base sans ajouter de règle font-family sur le body.

Sans Preflight, ce point de départ peut être utile :

```css
body {
  margin: 0;
  font-family: system-ui, sans-serif;
}
```

Ce fragment illustre les styles de base ; il ne fait pas partie du pattern thème.

## Option 1. Styles globaux — src/app.css

Dans ton projet avec Tailwind v4, ajoute ce bloc à src/app.css après ton import Tailwind existant :

```css
@layer base {
  button { cursor: pointer; }
  a:focus-visible, button:focus-visible {
    outline: 3px solid currentColor;
    outline-offset: 4px;
  }
}
```

Le curseur « main » est un choix facultatif. focus-visible affiche un indicateur lorsque le navigateur le juge nécessaire, notamment au clavier. currentColor reprend la couleur du texte ; vérifier son contraste avec le fond en clair et en sombre. Le décalage de 4 px éloigne le contour de l’élément.

Importe src/app.css une seule fois depuis ton layout SvelteKit (`import '../app.css';` dans le script de src/routes/+layout.svelte). La couche base permet aux utilitaires Tailwind de personnaliser ces règles. Le focus global cible les liens et boutons ; les autres contrôles conservent leur indicateur natif, à vérifier également au clavier.

## Option 2. Directement dans les composants

On peut gérer le curseur et le focus directement avec les utilitaires Tailwind :

```svelte
<button
  type="button"
  class="cursor-pointer focus-visible:outline-solid focus-visible:outline-3 focus-visible:outline-current focus-visible:outline-offset-4"
>
  Une action
</button>
```

Les classes focus-visible:* conviennent aussi aux liens. Choisir une base globale, des classes locales ou une base avec des adaptations locales. Ne pas supprimer un indicateur de focus sans alternative visible. Le bouton de la [recette thème](theme.md) peut bénéficier de cette règle globale ou recevoir ces classes, sans JavaScript supplémentaire.

## Réutiliser et vérifier

Choisis les règles qui conviennent à ton projet ; aucun fichier CSS séparé n’est imposé. Parcourir les liens et boutons avec Tab et Maj+Tab, vérifier le contour dans les deux thèmes et le focus des contrôles natifs.

Références : [Preflight](https://tailwindcss.com/docs/preflight), [polices](https://tailwindcss.com/docs/font-family) et [focus-visible](https://tailwindcss.com/docs/hover-focus-and-other-states#focus-visible).
