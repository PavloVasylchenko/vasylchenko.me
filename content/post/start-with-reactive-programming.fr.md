---
title: "Bien démarrer avec la programmation réactive"
date: 2022-08-04T22:00:00Z
categories:
  - "Manuals"
draft: false
---

Aujourd’hui, je souhaite parler des flux réactifs.
Cette approche est devenue très populaire ces dernières années. Spring permet de l’utiliser à la place du modèle fondé sur les servlets.
De plus en plus de bibliothèques clientes proposent une API réactive, tandis que les bases de données acceptent des connexions par R2DBC et des pilotes réactifs.
J’utilise moi aussi Project Reactor de plus en plus souvent dans mes modules.

Les flux réactifs sont parfois comparés aux Java Streams, car les deux technologies sont liées à la programmation fonctionnelle et présentent une syntaxe similaire.
Cependant, un `Stream` classique ne peut être consommé qu’une seule fois ; les développeurs renvoient donc rarement une valeur telle que `Stream<String>` depuis une méthode.
Ils placent généralement toute la chaîne de traitement dans une seule fonction. C’est pratique pour de petites transformations, mais les appels restent finalement bloquants.

Il y a quelques mois, j’ai suivi un cours Udemy consacré à Project Reactor afin d’organiser mes connaissances et de découvrir de nouvelles idées.
Le contenu s’est révélé intéressant et utile, en particulier pour les développeurs qui découvrent l’approche réactive.

J’ai ensuite réalisé une version du « Jeu de la vie » reposant sur des flux réactifs.
C’est un excellent cas d’usage : la source peut produire une séquence illimitée, puis le consommateur décide du nombre d’éléments à traiter.
Le code est disponible dans le dépôt [GameOfLife](https://github.com/PavloVasylchenko/GameOfLife).

Le projet comprend plusieurs éléments :

- `CellState`, une énumération indiquant si une cellule est vivante ou morte ;
- `Game`, qui contient la logique principale et le générateur des cycles ;
- `UI`, chargé d’afficher la grille dans la console.

La grille est représentée par un tableau bidimensionnel `CellState[][]`.
Une seule méthode publique reçoit l’état initial et renvoie un `Flux` d’états suivants calculés à partir de celui-ci :

```java
Flux<CellState[][]> game(CellState[][] initialField)
```

La méthode ne fixe volontairement aucune limite : c’est au client de choisir le nombre d’itérations.
Le flux est construit en deux étapes :

1. `Mono.just(initialField)` place la grille initiale en tête de la séquence.
2. `.concatWith(generate)` ajoute le générateur des générations suivantes.

```java
Flux<CellState[][]> generate = Flux.generate(() -> initialField, (state, sink) -> {
    CellState[][] iterate = iterate(state);
    sink.next(iterate);
    return iterate;
});
```

Le générateur calcule une nouvelle grille à partir de la précédente, l’envoie au `Flux` courant et la conserve comme état pour l’étape suivante.

Le test illustre l’utilisation de cette logique :

```java
@Test
public void printerTest() {
    new Game().game(getGliderField())
            .take(5)
            .doOnNext(UI::printState)
            .subscribe();
}
```

La partie démarre avec un motif « planeur », limite le flux à cinq éléments et affiche chaque grille.
Le générateur n’est exécuté que quatre fois, car l’état initial constitue déjà le premier élément.
Pour produire davantage de générations ou ajouter un traitement, il suffit d’insérer de nouveaux opérateurs dans le `Flux` existant.
