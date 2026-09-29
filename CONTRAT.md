# Le contrat d'accroche

Les templates fa-lab sont réécrits de zéro, mais ils gardent **à côté de leurs
propres classes** celles de ModernBB dont dépendent le JavaScript de
Forumactif et les plugins (`fa-feed`, `fa-updates`, et ceux des autres).

C'est ce qui permet à un plugin écrit pour ModernBB de fonctionner tel quel sur
un forum habillé par fa-lab — et à un essai fait sur fa-lab de valoir pour un
forum ModernBB ordinaire.

**Règle :** on n'ajoute pas de classe « à la ModernBB » qui n'existe pas dans
ModernBB, même si elle aiderait un plugin. Sinon un plugin qui marche ici
pourrait échouer ailleurs sans qu'on le voie.

---

## Ce que chaque template garde

### `viewtopic_body` — lu par fa-feed

| élément | pourquoi |
|---|---|
| `.post` sur chaque message, `#p{ID}` | le conteneur d'un message |
| `post--{ID}` dans la classe | l'identifiant du message ; sert aussi à `showHiddenMessage()` |
| `.postbody` › `.content` | le corps du message ; `resize_images()` le cible |
| `.postprofile`, `.postprofile-avatar[data-id]` | le profil |
| `.fa_like_div`, `.rep-button[data-href][data-href-rm]` | le « j'aime » : Forumactif branche son script dessus |
| `id="post_mq{TOPIC_ID}_{ID}"` | la citation multiple |
| `.signature_div`, `.dd_award`, `.award_more` | signatures et récompenses |
| `.pagination`, `.topic-actions` | la pagination |
| `#forum_rules.post` | le règlement de section — c'est un `.post` chez ModernBB aussi |

### `posting_body` — utilisé par le commentaire de fa-feed

| élément | pourquoi |
|---|---|
| `form[name="post"]`, `action="{S_POST_ACTION}"` | le formulaire que fa-feed remplit |
| `textarea#text_editor_textarea[name="message"]`, `#textarea_content` | SCEditor s'y greffe |
| `input[name="post"]` | le bouton d'envoi |
| `{ERROR_BOX}` | où Forumactif écrit ses refus |
| tous les `name` des champs, sans exception | c'est ce que le serveur lit |
| `#post_dice`, `#list_dice`, `#dice_to_del`, `#username`, `#add_username`, `#find_user`, `#find_username` | scripts des dés et de la messagerie |
| `.forum-hideable` + `.panel` venant de `{POLLBOX}` | rangés dans un volet par le template, sans renommer |

### `profile_advanced_body`

| élément | pourquoi |
|---|---|
| `dl[id^="field_id"]` › `dd` › `.field_uneditable` / `.field_editable` | l'édition en ligne des champs, par emboîtement |
| `.invisible` | ce même script masque par cette classe ; la feuille la masque |
| `#tabs`, `li.activetab` | les onglets générés par `{tab.TAB}` |
| `.followBtn`, `doFollowAction()` | suivre un membre |

### `topics_list_box` — lu par fa-updates

| élément | pourquoi |
|---|---|
| `li.row` par sujet | la ligne que fa-updates remonte depuis le lien |
| `a.topictitle` | le lien du sujet |
| `.lastpost`, `.lastpost-avatar`, `.lastpost-infos` | le dernier message |
| `<dfn>{L_LASTPOST}</dfn>` dans `.lastpost` | présent chez ModernBB, donc dans le texte que lisent les plugins ; masqué à l'écran |
| les blocs `multi_selection`, `single_selection` | le même template sert à la modération |

### `index_box`

| élément | pourquoi |
|---|---|
| `a.forumtitle`, `.lastpost`, `.lastpost-avatar` | comme ModernBB |
| `.forabg`, `.topiclist.forums`, `li.row` | idem |
| `data-lien`, `data-retrait` sur `li.fal-forum` | propres à fa-lab : régler une section depuis la feuille, repérer un sous-forum en ligne |

### `overall_header` / `overall_footer_*`

| élément | pourquoi |
|---|---|
| `<body id="modernbb">` | c'est par là qu'on reconnaît la version du forum |
| `#page-header`, `#page-body`, `#main-content`, `#page-footer` | cibles courantes des scripts et des plugins |
| `#modernbb-nav-menu.navbar` | la navigation ; `{GENERATED_NAV_BAR}` dans un `li` |
| `#page-footer .footer-home` | le bouton des notifications push s'y accroche |
| le `li.rightside` ouvert à la fin de `overall_footer_begin` | Forumactif y insère ses mentions obligatoires |
| la feuille `display: … !important` sur le pied de page | exigée par Forumactif |

---

## Les emplacements des plugins

Dans `overall_footer_end`, donc sur toutes les pages : **le dock**, une rangée
d'onglets au bord droit de l'écran. Chaque onglet ouvre le panneau de son
plugin, par-dessus la page ; un seul panneau est ouvert à la fois, il se ferme
par le ×, par Échap ou par son onglet, et le panneau ouvert est retenu d'une
page à l'autre.

```html
<div class="fal-dock" id="fal-dock">
  <div class="fal-dock__onglets">
    <button class="fal-dock__onglet" aria-controls="fal-dock-feed" aria-expanded="false">Le fil</button>
  </div>
  <section class="fal-dock__panneau fal-categorie" id="fal-dock-feed"
           data-plugin="FAFeed" data-config="FA_FEED_CONFIG" hidden>
    <header class="fal-categorie__entete">…<button class="fal-dock__fermer"></button></header>
    <div class="fal-dock__corps"><div id="fa-feed"></div></div>
  </section>
</div>
```

- **Un plugin absent n'a pas d'onglet.** Au chargement, un panneau dont l'objet
  `data-plugin` (`FAFeed`, `FAUpdates`) n'existe pas est retiré avec son onglet,
  et le dock entier s'il n'en reste aucun. Un forum sans plugins n'a donc rien à
  retirer.
- **Les scripts des plugins se chargent après le dock**, à la fin de
  `overall_footer_end`.
- **`manuel: true`** dans la configuration du plugin : le dock ne le lance qu'à
  la première ouverture de son panneau. Rien n'est lu tant que personne n'ouvre
  l'onglet. Sans `manuel`, le plugin se lance au chargement, panneau fermé ou
  non.
- Le titre du panneau est celui du template : on laisse le `titre` du plugin à
  `null` pour ne pas l'écrire deux fois.
- Réglages : `--fal-dock-largeur` (22rem). Le panneau prend la hauteur de son
  contenu, jusqu'à celle de l'écran ; au-delà, son corps défile.

Un autre plugin s'ajoute en recopiant un onglet et un panneau, avec son propre
`data-plugin`.

---

## Ce que les plugins ne trouvent pas — même sur ModernBB d'origine

Relevé le 28 septembre 2026, et identique sur un ModernBB non
modifié puisque le balisage concerné est conservé tel quel :

- **fa-feed, `balisage.date`** : ModernBB met la date dans `.topic-date`, que
  la liste par défaut (`a.post_date`, `.postdetails .date`, `.date`) ne
  contient pas. La date du message reste vide.
- **fa-updates, dernier posteur** : dans `.lastpost`, ModernBB écrit l'auteur
  *avant* la date. Le découpage « tout jusqu'à l'heure = la date » avalait donc
  l'auteur. Corrigé dans fa-updates le 29 septembre 2026 : le posteur est lu dans
  son lien de profil, la date dans ce qui reste.

C'est aux plugins de s'adapter, pas aux templates : les corriger ici ne
vaudrait que pour fa-lab.
