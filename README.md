# fa-lab

Une base de thème pour Forumactif (version **ModernBB**) : des templates
réécrits en HTML actuel, une feuille qui part du mobile, et des réglages
séparés du code.

Ce n'est pas un thème fini : c'est ce sur quoi on en construit un. La base est
volontairement sobre ; l'habillage vient d'un thème (`themes/`) ou de ses
propres réglages.

> **État : 0.1, en cours.** Dix templates sont écrits — l'accueil, les
> sections, les sujets, la rédaction et le profil. Les autres pages (recherche,
> liste des membres, messagerie…) gardent le balisage de ModernBB, rendu
> lisible par la feuille.

---

## Les fichiers

| fichier | à quoi il sert |
|---|---|
| `templates/*.html` | un fichier par template Forumactif, à coller tel quel |
| `fa-lab.css` | la structure. Rien à y modifier. |
| `config.css` | **les réglages** : toutes les variables, en commentaire |
| `themes/bureau.css` | un bureau pixel : fenêtres à barre de titre sur un damier, pastel le jour, ardoise la nuit |
| `themes/editorial.css` | un habillage tout fait : typographie de magazine, papier et encre |
| `CONTRAT.md` | les classes que les templates gardent pour les plugins et Forumactif |

---

## Installation

1. **Désactiver le CSS de base** : PA → Affichage → Couleurs → CSS principal →
   « Désactiver le CSS de base : Oui ». Mettre aussi « Optimiser votre CSS »
   sur Non : l'optimiseur ne connaît pas `light-dark()` ni `:has()`.

2. **Coller les templates** : PA → Affichage → Templates → Général. Pour
   chacun, remplacer le contenu, Enregistrer, puis publier (la coche verte).
   Les publier **tous ensemble** : l'en-tête et les pieds de page
   s'emboîtent, un seul publié casse la page.

   Général : `overall_header` · `overall_footer_begin` · `overall_footer_end` ·
   `index_body` · `index_box` · `viewforum_body` · `topics_list_box` ·
   `viewtopic_body`

   Poster & Messages privés : `posting_body`

   Profil : `profile_advanced_body` — c'est lui qui s'affiche quand le profil
   « avancé » est actif, ce qui est le réglage par défaut ; `profile_view_body`
   ne sert pas.

3. **La feuille** : dans PA → Affichage → Couleurs → CSS principal, coller
   `fa-lab.css`, puis un thème si on en veut un, puis ses réglages tirés de
   `config.css`. Un `@import` de polices doit être tout en haut.

4. **Les réglages de Forumactif** qui vont avec :
   - PA → Affichage → Page d'accueil → Structure et hiérarchie :
     « Séparer les catégories sur l'index : **Moyen** » (sinon les catégories
     n'ont pas de titre, et les sous-forums ne s'affichent pas en liens sous
     leur section), « Afficher les liens vers les sous-forums : Oui »,
     « Afficher les avatars dans la colonne Derniers messages : Oui » ;
   - PA → Affichage → Page d'accueil → Généralités → « Afficher la liste des
     membres connectés au cours des 24 dernières heures : Oui » ;
   - PA → Affichage → Templates → Version mobile → « L'adresse de votre forum
     dirige vers : Version web » — sinon les téléphones reçoivent la version
     mobile de Forumactif, qui n'utilise aucun de ces templates.

Pour revenir en arrière : « Valeur par défaut » sur chaque template, et
réactiver le CSS de base.

---

## Ce que la base apporte d'office

- **Les sous-forums** en pastilles sous leur section. Avec un index non
  compressé, ils s'affichent en lignes, décalées vers la droite.
- **Une illustration par section**, de deux façons :
  - l'image réglée dans le panneau (Catégories et forums → la section →
    Adresse de l'image) est reprise automatiquement, mais elle remplace
    l'icône d'état : on ne voit plus s'il y a du neuf ;
  - une ligne de CSS par section, qui garde l'icône d'état et accepte aussi une
    couleur ou un dégradé :

    ```css
    :root { --fal-illustrations: block; }
    .fal-forum[data-lien^="/f5-"] { --fal-illustration: url(https://…); }
    ```
- **L'avatar du dernier posteur**, à l'accueil et dans la liste des sujets.
- **Un « qui est en ligne » en chiffres** : en ligne, record, messages,
  membres, dernier inscrit, puis les connectés du moment et des dernières 24 h
  en pastilles, et la légende des groupes. Les libellés sont dans
  `index_body`, attribut `data-libelle`.

---

## Jour / nuit

L'en-tête porte un bouton à trois positions : **auto** (le réglage de
l'appareil), **jour**, **nuit**. Le choix est retenu dans le navigateur du
visiteur. Un petit script, tout en haut de `overall_header`, pose avant
l'affichage deux attributs sur `<html>` :

- `data-fal-mode` : le choix (`auto`, `jour`, `nuit`) ;
- `data-fal-rendu` : ce qui s'affiche (`jour` ou `nuit`).

Les couleurs de la base, écrites en `light-dark()`, suivent d'elles-mêmes. Un
thème qui change aussi de polices ou de décor s'accroche à
`:root[data-fal-rendu="nuit"]` : c'est ce que fait `bureau.css`.

Pour un forum sans mode sombre : `:root { --fal-apparence-affichage: none; }`
et `:root[data-fal-rendu] { color-scheme: light; }`.

---

## Ce qu'il faut savoir en l'écrivant

- **Pas de balise dans le CSS**, même en commentaire : Forumactif supprime tout
  ce qui ressemble à `<li>` en enregistrant la feuille.
- **Pas d'emoji dans le CSS** : l'encodage de stockage les remplace par `????`.
- **Pas d'accolades vides dans un template** : `catch (e) {}` perd ses `{}`,
  que Forumactif prend pour une variable vide, et le script casse. Écrire
  `catch (e) { void e; }`.
- **Pas de commentaire `//` dans un script de template** : au rendu,
  Forumactif supprime les retours à la ligne, et le commentaire avale tout ce
  qui suit. Seulement `/* … */`, et un `;` à chaque fin d'instruction.
- **La feuille CSS du panneau est limitée à 64 000 caractères environ.**
  Au-delà, Forumactif garde l'ancienne sans rien dire. `fa-lab.css` compactée
  en pèse déjà 45 000 : elle se charge par jsDelivr, le panneau ne garde que le
  thème et les réglages du forum.
- **Pas de `{VARIABLE}` dans un commentaire**, ni HTML ni JavaScript :
  Forumactif la remplace quand même. `{POLLBOX}` dans un commentaire de script
  y injecterait tout le formulaire de sondage.
- **`{catrow.tablehead.L_FORUM}` contient déjà un `h2`** quand les catégories
  sont séparées. L'entourer d'un autre titre le vide.
- La barre d'outils Forumactif pose une marge en haut de page, prévue pour une
  barre fixe ; `fa-lab.css` l'annule.

---

## Les versions

La feuille sera servie par jsDelivr **à une version figée** :

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/LisaThoa/fa-lab@0.1.0/fa-lab.css" />
```

Jamais `@main` : chaque changement poussé arriverait sur tous les forums sans
prévenir, y compris ceux qui cassent quelque chose.
