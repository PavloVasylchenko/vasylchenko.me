---
title: "Bien démarrer avec Hugo"
date: 2020-10-19T19:00:00Z
categories:
  - "Manuals"
draft: false
---

Créer un blog avec Hugo.
Ces dernières années, les sites statiques sont devenus très populaires.
Cela s’explique probablement par leur simplicité de maintenance et par le grand nombre de plateformes permettant de les publier.
Alors qu’il fallait autrefois manipuler directement du HTML pour créer un site statique, la connaissance de Markdown suffit aujourd’hui pour la plupart des besoins.
Les sites Hugo conviennent très bien aux pages personnelles dont le contenu ne doit pas être mis à jour fréquemment.
Hugo est un ensemble d’outils qui génère un site statique à partir de fichiers Markdown et de modèles.
J’apprécie également l’existence d’images Docker prêtes à l’emploi : elles permettent de créer et de compiler le site sans installer Go, Hugo ni d’autres composants sur la machine.

Commençons.
Si vous utilisez Linux avec Docker installé ([docker.com](https://www.docker.com)), quelques étapes suffisent.
Créez un site vide dans le répertoire `vasylchenko.me` avec la commande suivante :

```bash
docker run --rm -it -v $(pwd)/:/src klakegg/hugo:ext new site vasylchenko.me
```

Entrez ensuite dans le dossier avec `cd vasylchenko.me`.
Initialisez alors un dépôt Git pour assurer le suivi des versions :

```bash
git init
```

Il est maintenant temps de choisir un thème.
Vous trouverez une sélection de thèmes populaires sur [themes.gohugo.io](https://themes.gohugo.io).
Pour ce site, j’ai par exemple choisi le thème [Devise](https://themes.gohugo.io/devise/).
Ouvrez la page du thème, puis exécutez la commande suivante :

```bash
git submodule add https://github.com/austingebauer/devise themes/devise
```

Cette commande ajoute le dépôt du thème au répertoire `themes` sous forme de sous-module Git.

Nous disposons désormais du dossier du site `vasylchenko.me` et de celui du thème `vasylchenko.me/themes/devise`.
Il faut ensuite indiquer à Hugo quel thème utiliser dans la configuration.
Vous pouvez modifier le fichier manuellement ou lancer :

```bash
echo 'theme = "devise"' >> config.toml
```

Il ne reste plus qu’à générer le site et à vérifier le résultat :

```bash
docker run --rm -it -v $(pwd):/src klakegg/hugo:ext
```

Une fois la compilation terminée, ouvrez le dossier `vasylchenko.me/public`, puis le fichier `index.html` dans votre navigateur.
Si le rendu correspond à vos attentes, vous pouvez envoyer le contenu de ce dossier vers un hébergement statique.
