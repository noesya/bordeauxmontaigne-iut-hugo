# IUT Bordeaux Montaigne

Pour installer Hugo `brew install hugo`


Pour lancer le site
```
yarn
hugo server
```

## Home page

Il faut un fichier _index.html à la racine du dossier content, qui sert de home page, avec la body class `page__home`.

```
content
|   _index.html
```

Il faut remettre la description sans HTML, pour le SEO, mettre le contenu HTML dans le corps du fichier, par souci de cohérence. Le breadcrumb_title permet de gérer le breadcrumb de façon unifiée.
```markdown
---
title: L’IUT Bordeaux Montaigne, le plus court chemin pour aller loin
breadcrumb_title: Accueil
image: "29f2e051-1298-48d4-9e21-5badbd606d30"
description: >
  L'Institut Universitaire de Technologie Bordeaux Montaigne, le plus court chemin pour aller loin
---
L’IUT Bordeaux Montaigne,<br><em>le plus court chemin pour aller loin</em>
```

## Branches ou feuilles ?

Il y a 2 façons de structurer les contenus : en branches (ou sections) et en feuilles (ou pages).

### Branches
Avec des fichiers _index.html:
```
content
└───programs/_index.html
│   └───bachelor-universitaire-de-technologie/_index.html
│       └───carrieres-sociales/_index.html
│           └───parcours-animation-sociale-et-socioculturelle/_index.html
```
Les branches sont pensées pour structurer des contenus, et sont donc rendues avec le template "list.html".


L'affichage des enfants dans l'ordre se fait comme cela :
```
{{ range sort .Pages ".Params.position"  }}
  <a href="{{ .Permalink }}">{{ .Title }}</a>
{{ end }}
```

### Feuilles
Avec des fichiers nommés:
```
content
└───pages
│   │   notre-institut.html
│   └───notre-institut
│       │   consignes-de-securite.html
│       │   equipe-pedagogique.html
```
Les feuilles sont des objets indépendants, rendues avec le template "single.html"

Pour afficher les enfants, il faut ajouter une propriété child_pages (le nom est arbitraire):
```markdown
---
...
child_pages:
  - "/notre-institut/presentation-de-liut/"
  - "/notre-institut/consignes-de-securite/"
...
---
```

L'affichage se fait comme cela :
```
  {{ range .Params.child_pages }}
    {{ $page := partial "utils/GetPageByUrl.html" . }}
    <div class="col">
      <a href="{{ $page.Permalink }}">{{ htmlUnescape $page.Title }}</a>
    </div>
  {{ end }}
```

### Hybride
Dans l'idée de départ de Hugo, on fait des branches quand on veut grouper des feuilles, et donc on mélange les deux.
On peut le faire, ce qui est très idiomatique, mais cela oblige à ranger différemment les objets qui ont des enfants et ceux qui n'en ont pas, ce qui rend l'algorithme d'export plus compliqué :
```
content
└───pages/
│   └───notre-institut/_index.html
│       │   consignes-de-securite.html
│       │   equipe-pedagogique.html
└───programs/_index.html
│   └───bachelor-universitaire-de-technologie/_index.html
│       └───carrieres-sociales/_index.html
│           │   parcours-animation-sociale-et-socioculturelle.html
```

### Arbitrage
Dans les 3 cas, il faut spécifier les urls dans le frontmatter des contenus.
D'après mes tests, on ne peut pas s'appuyer sur une génération de permalinks récursifs à partir des slugs.
Les sections utilisent toujours les noms de fichiers si elles n'ont pas d'url spécifiée, on ne peut pas leur attribuer de format de permalink dans la config.
Les permalinks ne fonctionnent que pour les pages.


La logique de branche est plus native pour lister les enfants, elle utilise l'objet .Pages, et on ajoute la position pour gérer l'ordre.
Elle implique que les objets sont tous rendus par le template list, les feuilles étant considérées comme des listes vides, ce qui est curieux mais pas très grave.
La logique de feuille avec child_pages utilise un partial maison (utils/GetPageByUrl), qui itère sur l'ensemble des pages.
C'est certainement sous-optimal en termes de performance.


Le choix pur feuilles paraît sans bénéfice.
Le choix pur branches est simple en termes de génération, avec un manque d'élégance.
Le choix hybride est le plus compliqué à générer et le plus élégant du point de vue d'Hugo.


En attente de décision définitive, tout branches.

## Pages

Il faut changer l'architecture vers un système de branches :
```
content
└───pages
│   └───notre-institut/_index.html
│       └───consignes-de-securite/_index.html
│       └───equipe-pedagogique/_index.html
```

Il faut aussi des slashs finaux aux URLS, c'est la pratique adoptée par Hugo (https://discourse.gohugo.io/t/hugo-support-for-urls-without-a-trailing-slash/6763)


Il serait préférable de remettre le text dans le corps des documents et pas dans le frontmatter, puisque nous n'allons plus récupérer les contenus existants sur Github.


```markdown
---
title: "Plan d’accès et contact"
url: "/notre-institut/plan-dacces-et-contact/"
position: 1
categories:
  - "a-la-une"
description: >
  Où sommes nous ? ATTENTION : nous ne sommes pas sur le site du Campus
---
Où sommes nous ?<br><br>  <strong>ATTENTION : nous ne sommes pas sur
```

Suppression d'identifier et parent qui ne sont pas nécessaires.
On garde position pour mettre les pages enfants dans le bon ordre.

## Posts

L'arborescence peut être reprise telle quelle sans problème.

Un fichier _index.html doit être ajouté à la racine pour matérialiser la liste des actualités.

Les fichiers doivent être harmonisés,
```markdown
---
title: "Une bourse ? Un logement ? Constituez dès maintenant votre dossier social étudiant (DSE)"
date: 2021-01-25 15:02:00 +0100 UTC
slug: "constituez-votre-dossier-social-etudiant"
weight: 1
authors:
  - "hbeneyrol"
categories:
  - "a-la-une"
image: "d97ed0d8-27f6-4c29-8bc3-aac021e6308a"
description: >
  N’attendez pas les résultats de vos examens ou votre affectation sur Parcoursup...
---
<img width="480" height="230" src="https://www.iut.u-bordeaux-montaigne.fr/wp-content/uploads/2021/01/Demande_DSE_2021.png"> N’attendez pas les résultats de vos examens ou votre affectation sur Parcoursup ...
```

Weight est utilisé pour épingler les articles, à 0 l'article est dans le flux normal, à 1 il est mis en avant.

## Catégories

Il faut changer l'architecture vers un système de feuilles :
```
content
└───categories/_index.html
│   └───a-la-une/_index.html
│   ...
```

Les catégories ressemblent à ça :
```
---
title: "À la une"
slug: "a-la-une"-
description: >
  Flash info publiés sur la page d'accueil
---
```

Quid des catégories nested? A tester.

## Auteurs

Les auteurs sont déclarés comme une taxonomie dans le fichier de config :


Les contenus sont organisés comme les catégories :
```
content
└───authors
│   │   _index.html
│   └───justin-puyo/_index.html
│   ...
```

Chaque auteur est défini de cette façon, attention à mettre le prénom avant le nom :
```
---
title: "Justin Puyo"
slug: "justin-puyo"
first_name: "Justin"
last_name: "Puyo"
biography: >
  Lorem ipsum dolor sit amet
---
```

Le rendu se fait avec les layouts authors/list.html et authors/term.html.

## Formations

Les formations sont gérées en sections Hugo, ce qui permet de les lister en arbre.
```
content
└───programs/_index.html
│   └───bachelor-universitaire-de-technologie/_index.html
│       └───carrieres-sociales/_index.html
│           └───parcours-animation-sociale-et-socioculturelle/_index.html
```

La position est maintenue dans le frontmatter, pour trier les formations d'un même niveau.

```
---
title: "BUT Animation sociale et socioculturelle"
url: /programmes/bachelor-universitaire-de-technologie/carrieres-sociales/parcours-animation-sociale-et-socioculturelle/
continuing: false
level: bachelor
ects:
position: 1
accessibility: >

contacts: >
   <strong>Responsable pédagogique : </strong><br>Francesca Lynn...
duration: >

evaluation: >

objectives: >
  Le parcours « Animation sociale et socioculturelle » forme des futurs professionnels ...
opportunities: >
  Le B.U.T. peut déboucher sur une insertion professionnelle ou sur une poursuite ...
other: >

pedagogy: >
  Les 3 années du programme national comportent 1800 heures de formation sur 6 semestres, ...
prerequisites: >
   La formation est ouverte aux élèves de terminale (toutes spécialités), aux titulaires ...
pricing: >

registration: >

---
```

## SEO

Attention, à l'heure actuelle on utilise l'image originale pour le SEO, il faut utiliser une image redimensionnée.

## Breadcrumb

Rien à signaler, tout fonctionne avec un partial.

## Menu

Attention, mettre des trailing slashs partout.
Si les liens sont externes, mettre un target blank.

## Medias

Je n'ai pas reconstitué les src-set, il faut le faire, avec la syntaxe d'Osuny
