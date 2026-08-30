---
title: "De l’idée à Google Play : comment j’ai créé un jeu avec l’IA pendant mes vacances"
date: "2026-08-30T00:00:00Z"
draft: false
description: "Comment le souvenir d’un petit jeu de stratégie sur navigateur est devenu une expérience avec plusieurs modèles d’IA, un prototype fonctionnel et mon propre jeu Android."
slug: "from-flash-game-to-android-game"
categories:
  - "Game Development"
  - "Artificial Intelligence"
tags:
  - "Synaptic Front"
  - "AI-assisted development"
  - "Indie Game"
series: "Synaptic Front with AI"
series_order: 1
---

Quand j’étais au lycée, nous avons enfin eu à la maison une connexion Internet assez rapide pour l’époque. Les jeux sur navigateur ont alors trouvé leur place dans mon quotidien : aucune installation, un chargement rapide, et juste ce qu’il fallait pour occuper une courte pause. Pendant qu’une archive se décompressait ou qu’un logiciel s’installait, je pouvais ouvrir un nouvel onglet, jouer une petite partie, puis reprendre ce que je faisais quelques minutes plus tard. Parmi tous ces jeux Flash, un jeu de stratégie m’est particulièrement resté en mémoire, même si j’ai fini par en oublier le nom.

Le terrain de jeu était composé de systèmes reliés par des lignes, le long desquelles se déplaçaient des flottes. L’interface indiquait immédiatement à qui appartenait chaque nœud, où envoyer ses forces et d’où approchait l’adversaire. Chaque décision modifiait l’équilibre : une attaque renforçait un front, mais affaiblissait le système de départ, tandis qu’un nœud conquis pouvait transformer une zone sûre en nouvelle frontière. Les règles étaient simples et leurs conséquences apparaissaient en quelques secondes. C’est précisément ce qui rendait même les parties courtes si tendues.

Flash a fini par disparaître, les ordinateurs et les habitudes ont changé, et le nom du jeu n’a survécu ni dans mes favoris ni dans ma mémoire. J’ai essayé plusieurs fois de le retrouver à partir de sa description, sans découvrir autre chose que des projets similaires. L’idée, elle, était restée très nette : un réseau de nœuds connectés, des déplacements sur des itinéraires définis et une tension née de quelques règles faciles à comprendre. Des années plus tard, ce souvenir est devenu le point de départ de Synaptic Front.

#### D’un souvenir à mon propre jeu

L’idée de créer un jeu semblable m’est venue plus d’une fois, mais elle est longtemps restée dans la liste des projets auxquels je ne trouvais jamais le temps de m’attaquer. Pour réaliser un premier prototype, il fallait choisir une technologie et comprendre la boucle de jeu, l’affichage de la carte, les animations et les commandes. Même si la mécanique s’avérait finalement peu intéressante, il aurait d’abord fallu consacrer du temps à des outils inconnus. L’expérience coûtait trop cher : plusieurs semaines de travail pouvaient aboutir à une conclusion que je voulais obtenir en une ou deux soirées.

J’ai aujourd’hui plus de quinze ans d’expérience en développement Java. J’ai travaillé sur des applications d’entreprise et des systèmes serveur à forte charge ; une architecture complexe ou un projet volumineux ne m’effraient donc pas. Mais je n’avais jamais créé de jeu. Android, Kotlin et Compose ne faisaient pas non plus partie de mon travail habituel. Je savais programmer, mais cette idée précise m’obligeait tout de même à partir à la découverte d’un domaine inconnu.

> **« L’IA n’a pas inventé ce jeu : elle m’a aidé à enfin commencer à le créer. »**

Ces derniers temps, en revanche, je travaille beaucoup avec des outils d’IA et je sais désormais assez bien dans quels cas ils font réellement gagner du temps. J’ai décidé d’associer cette expérience à mon ancienne idée pour vérifier enfin si elle pouvait donner un jeu. Je voulais arriver au plus vite à une version que je pourrais lancer et juger moi-même, sans passer plusieurs semaines à simplement découvrir une nouvelle stack. Claude, GPT, Kimi et Fable, qui venait tout juste de sortir, ont participé à l’expérience.

Un jeu convenait mieux à cette vérification qu’un exercice abstrait. Je me souvenais précisément de la sensation recherchée, je comprenais les règles essentielles et je voyais aussitôt si chaque nouvelle version se rapprochait de mon intention. La question n’était pas de savoir combien de code un modèle d’IA pouvait écrire, mais si le jeu obtenu serait agréable à jouer.

#### Une spécification comme référence commune

Le travail a donc commencé par une description des règles, pas par la génération de code. J’ai préparé quelques idées de départ et demandé à Fable de les organiser dans une spécification Markdown. C’est ainsi qu’est né `SPEC.md`, premier document à décrire le concept général, la structure de la carte, la propriété des systèmes, la production des forces, le déplacement des flottes et les conditions de conquête. D’autres modèles l’ont ensuite analysé à tour de rôle. Ils précisaient les formulations, repéraient les ambiguïtés et proposaient des ajouts ; je conservais les changements utiles et supprimais ce qui éloignait le jeu de son idée initiale.

À mesure que le concept évoluait, un unique `SPEC.md` ne suffisait plus. Il a été rejoint par `VISUAL.md`, qui rassemblait les choix graphiques et les animations ; `PLOT.md`, consacré à l’intrigue des différents niveaux ; et `LORE.md`, qui contenait le canon général de l’univers. `CONTENT_PLAN.md` est venu ensuite pour transformer le récit et les idées de campagne en plan technique plus concret. Plus tard se sont ajoutés `ECONOMY.md`, sur l’évolution future de l’économie et des mécaniques associées, et `OPERATORS.md`, qui décrivait les héros opérateurs contrôlés par le joueur et leurs capacités.

Au début, cette structure séparait efficacement les différentes dimensions du projet, mais les documents ont vite grossi jusqu’à devenir eux-mêmes difficiles à lire. Je les ai divisés en fichiers thématiques plus petits et ajouté un `INDEX.md` pour s’y retrouver. Une règle, un élément narratif ou la description d’une capacité pouvait alors être consulté séparément, sans charger toute la matière accumulée. Cela s’est révélé utile pour moi comme pour les modèles : chaque tâche pouvait n’être accompagnée que de la partie pertinente de la documentation.

> **« Quand le code passait d’un modèle à l’autre, la spécification empêchait le projet de sombrer dans le chaos. »**

Ce système ne servait pas uniquement de description détaillée du futur jeu. Il séparait aussi les décisions déjà prises des suppositions propres à un modèle. Avec une simple demande comme « crée un jeu de stratégie avec des points et des lignes », un modèle comblera lui-même tous les vides : comportement de l’ennemi, vitesse des déplacements, calcul du résultat et bien d’autres détails. Deux implémentations ne différeront alors pas seulement par la qualité du code : elles constitueront en réalité deux jeux différents.

La documentation limitait les décisions arbitraires et permettait de ramener chaque modèle vers la version convenue du projet. Cela est devenu particulièrement utile lorsque le travail a commencé à circuler entre eux. Je n’avais plus à répéter tout le contexte : le modèle suivant recevait la bonne partie de la spécification, l’implémentation actuelle et une tâche précise. Les documents n’étaient toutefois pas un cahier des charges immuable. Après mes parties de test, j’ajustais les règles puis leur description afin que le code et l’intention restent alignés.

Une fois la spécification suffisamment détaillée, j’ai demandé aux modèles de produire les premières versions du jeu dans un fichier HTML unique. Ce choix répondait à un besoin de validation rapide, pas à l’architecture future. Le fichier s’ouvrait directement dans le navigateur, sans projet, compilation, dépendances ni infrastructure séparée. Il réunissait logique de jeu, interface et animation, si bien que le résultat de chaque itération devenait visible presque immédiatement.

L’expérience n’avait jamais vocation à être un benchmark rigoureux. Les modèles n’ont reçu ni prompts parfaitement identiques, ni limites fixes, ni système formel de notation. Les premières versions ont été créées indépendamment ; ensuite, les limites des abonnements et la nature des tâches ont fait circuler le travail de plus en plus souvent d’un outil à l’autre. Un modèle pouvait poser les fondations, le suivant corriger un problème, et un troisième proposer l’évolution d’une mécanique. En définitive, je comparais moins les modèles entre eux que les différentes manières d’organiser leur travail successif sur un même projet.

#### Le premier prototype et les cycles courts

La mécanique centrale du prototype restait compacte. La carte contenait des systèmes reliés par des routes, chacun produisant progressivement des unités. Le joueur et l’adversaire contrôlé par l’ordinateur envoyaient leurs flottes entre les nœuds voisins, conquéraient de nouvelles positions et tentaient de conserver celles qu’ils occupaient déjà. Pour réussir une attaque, choisir une cible faible ne suffisait pas : il fallait mesurer l’affaiblissement de la défense au départ de la flotte et déterminer si l’ennemi pourrait profiter du passage ainsi ouvert.

La première version convaincante est apparue lorsque la flotte sélectionnée a réellement suivi la ligne jusqu’au système indiqué. L’animation dépassait ce que j’attendais d’un premier prototype HTML, mais l’essentiel était ailleurs : je pouvais enfin évaluer la mécanique en action. Jusqu’alors, il n’existait qu’un souvenir, une spécification et des règles isolées. Désormais, je pouvais jouer un coup, en observer les conséquences et voir si la carte produisait la tension qui avait motivé toute l’expérience.

Le prototype a confirmé que le travail méritait d’être poursuivi, puis les cycles de développement courts ont commencé. Je précisais une règle dans la spécification, confiais une tâche circonscrite à un modèle, lançais la nouvelle version et disputais quelques parties. Si le comportement différait de mes attentes, l’écart devenait la tâche suivante. Ce processus apportait plus que la seule vitesse de génération : chaque changement conduisait rapidement à un résultat vérifiable.

Au fil des itérations, je pouvais modifier le rythme de production, la vitesse des flottes et l’équilibre entre attaque et défense, puis observer immédiatement l’effet des nouvelles valeurs. Peu à peu, j’ai compris quelles difficultés incitaient à chercher une solution et lesquelles semblaient simplement injustes. Mon vieux souvenir donnait la direction, mais les règles précises de Synaptic Front se sont formées pendant les tests.

Un éditeur de niveaux rudimentaire est devenu l’outil auxiliaire suivant. La première carte pouvait être décrite à la main dans les données, mais une véritable validation exigeait différentes configurations de nœuds et de routes. L’éditeur permettait de les créer plus vite et de vérifier si le jeu restait intéressant au-delà d’une seule disposition réussie. Il n’était pas destiné aux joueurs et n’exigeait aucune finition produit, mais il a nettement accéléré la boucle principale. Pour ce type d’outil interne, la génération par IA s’est montrée particulièrement rentable : un faible effort d’implémentation était amorti dès les itérations suivantes.

#### Du HTML à Android

Après plusieurs versions fonctionnelles, il était clair que l’idée avait passé son premier test. Une difficulté plus grande est alors apparue : dépasser le fichier HTML autonome. Android m’était plus familier qu’iOS ; j’ai donc choisi Kotlin et Compose pour la version suivante. Cette transition a tracé la frontière entre une expérience rapide et le développement d’une véritable application.

> **« HTML avait prouvé que l’idée fonctionnait ; Android devait la transformer en produit. »**

J’avais alors déjà essayé plusieurs modèles. Fable fut le premier : il venait de sortir et je voulais voir comment il traiterait une tâche réelle. D’après mon expérience, GPT-5.5, disponible à l’époque dans Codex, était moins performant ; j’utilisais donc à peine Codex. Claude est resté mon outil principal pendant les premières étapes du développement Android.

Dans le prototype web, logique, affichage et état pouvaient cohabiter, et une version ratée se remplaçait facilement en bloc. Le projet Android a introduit le cycle de vie de l’application, la persistance des données, la gestion de l’état, les tests et la nécessité de vérifier le comportement sur un appareil réel. Un écran fonctionnel demeurait un résultat important, mais ne prouvait plus que le produit était prêt. Chaque modification devait être évaluée au regard de la structure existante et de ses effets sur le reste de l’application.

La situation a changé avec l’arrivée de Sol. Le nouveau modèle travaillait avec bien plus d’assurance que 5.5 et à un niveau comparable à Fable ; j’ai donc résilié mon abonnement Claude et poursuivi le développement dans Codex. J’avais alors accumulé trois réinitialisations de limite, et l’abonnement Plus suffisait pour avancer sans me presser. J’ai construit l’essentiel de l’application à ce rythme : définir la tâche suivante, vérifier le résultat, puis continuer lorsque la limite redevenait disponible.

Comme Claude, Codex pouvait lancer lui-même l’émulateur Android, ouvrir l’application, prendre des captures et vérifier le résultat d’une modification. Si un écran s’affichait mal ou qu’un scénario échouait, le modèle pouvait constater le problème dans l’émulateur, revenir au code, le corriger et relancer la vérification. Pour Android, cela s’est révélé bien plus utile que la simple génération de fichiers : une grande partie du cycle entre modification et contrôle visuel se déroulait sans bascule manuelle entre les outils.

Kimi K3 est apparu vers le milieu du développement, et j’ai commencé à lui confier certaines tâches. La suite de l’application a donc été construite à plusieurs : certaines parties par Codex avec Sol, d’autres par Kimi K3. Les services atteignaient leurs limites à tour de rôle ; plutôt que d’attendre, je changeais de modèle. C’était aussi un nouveau test du processus : un nouvel outil saurait-il reprendre correctement le projet après le précédent ?

Chaque modèle commençait par lire et clarifier la documentation avant de modifier le code. Lorsqu’une nouvelle contrainte apparaissait ou qu’une autre décision était prise, elle retournait dans les documents et rejoignait le contexte commun. La transmission fonctionnait ainsi dans les deux sens : le modèle suivant recevait le savoir accumulé et laissait une description plus précise à son successeur. Progressivement, choisir un unique « meilleur » modèle a perdu son sens. La qualité de la documentation, la taille de la tâche et la possibilité de vérifier indépendamment le changement comptaient bien davantage.

#### L’évolution du rôle de l’IA à mesure que le projet grandit

Dans le premier prototype, le coût d’une erreur était faible. Une version médiocre pouvait être écartée après une seule session. Il était donc possible de confier de larges pans de l’implémentation aux modèles et d’explorer rapidement plusieurs variantes. Dans le projet Android, le même degré de confiance a commencé à ralentir le travail. La base de code grandissait, des liens apparaissaient entre les composants, et corriger un problème pouvait invalider une hypothèse dont dépendait une autre partie de l’application.

> **« Plus le projet grandissait, moins je pouvais confier du travail à l’IA sans réserve. »**

Au fil du projet, le rôle de l’IA est passé de la génération d’implémentations entières à une assistance d’ingénierie. Il devenait plus utile de discuter plusieurs solutions, de vérifier des hypothèses, de rechercher l’origine d’un écart et de préparer de petites modifications maîtrisées. Chaque étape importante exigeait toujours des tests et un contrôle manuel, et la décision finale restait celle de la personne qui comprenait le produit et répondait du résultat.

La synchronisation de la documentation avec le code a également pris une importance particulière. Lorsque je corrigeais un comportement à la main, conserver le changement dans le dépôt ne suffisait pas. La nouvelle règle ou décision d’architecture devait aussi apparaître dans les documents fournis aux modèles. Sinon, en quelques itérations, l’humain et l’IA travaillaient sur deux versions différentes du projet : le modèle réintroduisait une ancienne contrainte ou tentait de restaurer une décision abandonnée, et le temps se perdait à résoudre des contradictions évitables.

Cette expérience n’a pas montré que l’IA simplifie le développement de jeux ou dispense d’apprendre une nouvelle stack. Sa valeur se trouvait ailleurs : ces outils m’ont permis d’atteindre une première version testable avant que le coût d’entrée ne me fasse renoncer à l’idée.

Après le prototype HTML réussi, le problème a changé. Les doutes sur la mécanique ont cédé la place aux questions ordinaires du développement produit : comment maintenir le code, où placer les frontières de l’état, comment vérifier les modifications et comment préserver la connaissance des décisions prises. C’est ainsi qu’une expérience mêlant plusieurs modèles et des fichiers HTML autonomes est devenue peu à peu un projet Android, et que le souvenir d’un jeu Flash sans nom est devenu Synaptic Front.

> **Essayez Synaptic Front**
>
> Le jeu est déjà disponible sur [Google Play](https://play.google.com/store/apps/details?id=me.vasylchenko.synapticfront). Vous pourrez y voir ce qu’est devenue l’idée racontée ici et y jouer vous-même.
>
> *À suivre.*
