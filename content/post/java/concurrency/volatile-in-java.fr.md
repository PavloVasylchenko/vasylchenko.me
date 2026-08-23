---
title: "Comprendre le mot-clé volatile en Java"
date: 2024-06-17T10:00:00Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

Le mot-clé `volatile` de Java indique que la valeur d’une variable peut être modifiée par plusieurs threads. Lorsqu’une variable est déclarée `volatile`, les threads doivent observer sa valeur à jour au lieu de continuer à utiliser une ancienne copie conservée localement. Cette garantie est essentielle dans un environnement concurrent, où un thread pourrait autrement ignorer la modification effectuée par un autre.

**Pourquoi utiliser volatile ?**

Pour améliorer les performances, chaque thread et chaque processeur peuvent garder des valeurs dans des registres ou des caches. Un thread risque alors de travailler avec une donnée obsolète même si un autre a déjà écrit une nouvelle valeur. `volatile` impose des règles de visibilité : une écriture publie la modification et une lecture ultérieure récupère la valeur actualisée.

**Principales caractéristiques de volatile**

1. **Visibilité.** Les changements apportés à une variable `volatile` deviennent visibles pour les autres threads. L’écriture ne peut pas rester uniquement dans la vue locale de son auteur.
2. **Atomicité.** `volatile` ne rend pas les opérations composées atomiques. Par exemple, `counter++` comprend une lecture, une incrémentation et une écriture ; deux threads peuvent donc perdre une mise à jour.
3. **Ordonnancement.** Le Java Memory Model limite la réorganisation des lectures et écritures autour de `volatile`. Une écriture happens-before par rapport à une lecture ultérieure de la même variable dans un autre thread.

**Quand utiliser volatile ?**

1. Lorsqu’une variable simple est partagée par plusieurs threads.
2. Lorsqu’un lecteur doit voir la dernière valeur publiée par un autre thread.
3. Lorsque l’état évolue par une affectation unique et ne participe pas à une opération read-modify-write, check-then-act ou à un invariant complexe.

Les exemples courants sont un indicateur d’arrêt, un signal de disponibilité ou une référence vers une configuration immuable.

**Exemple d’utilisation de volatile**

```java
public class VolatileExample {
    private volatile boolean flag = false;

    public void writer() {
        flag = true;  // écriture dans une variable volatile
    }

    public void reader() {
        if (flag) {  // lecture de la variable volatile
            System.out.println("Flag is true!");
        }
    }

    public static void main(String[] args) {
        VolatileExample example = new VolatileExample();

        Thread writerThread = new Thread(() -> {
            try {
                Thread.sleep(1000); // simulation d'un traitement
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            example.writer();
            System.out.println("Flag set to true by writer thread");
        });

        Thread readerThread = new Thread(() -> {
            while (!example.flag) {
                // attente jusqu'à ce que le drapeau passe à true
            }
            example.reader();
        });

        writerThread.start();
        readerThread.start();
    }
}
```

Dans cet exemple, le champ `flag` est déclaré `volatile`. Le thread d’écriture attend une seconde puis lui affecte `true`. Le thread de lecture vérifie le drapeau dans une boucle et, grâce à la garantie de visibilité, détecte la nouvelle valeur. Il quitte alors la boucle et appelle `reader()`.

Sans `volatile`, le Java Memory Model n’oblige pas le lecteur à récupérer de nouveau la valeur partagée. En théorie, il pourrait continuer à voir `false` indéfiniment. La boucle d’attente active n’est pas non plus une bonne solution en production ; elle sert uniquement à illustrer clairement le problème de visibilité.

**Limites de volatile**

1. **Absence d’atomicité pour les actions composées.** Une incrémentation, une comparaison suivie d’une écriture ou la mise à jour cohérente de plusieurs champs exigent un autre mécanisme.
2. **Pas de protection pour un état complexe.** Si plusieurs valeurs doivent rester cohérentes, utilisez `synchronized`, `ReentrantLock` ou une structure de données thread-safe.
3. **Pas de coordination entre threads.** Pour attendre des événements, préférez `CountDownLatch`, les conditions d’un verrou, les files ou d’autres abstractions de haut niveau.

**Résumé**

`volatile` est un outil léger pour publier un état simple entre plusieurs threads. Il garantit la visibilité et un certain ordre mémoire, mais ne rend pas atomiques les opérations composées. Il convient souvent aux drapeaux et aux affectations indépendantes ; les compteurs et transitions complexes nécessitent des classes atomiques, des verrous ou des sections synchronisées. Choisir le bon mécanisme permet d’écrire un code concurrent correct et lisible.
