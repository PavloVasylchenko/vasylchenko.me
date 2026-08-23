---
title: "Les classes atomiques en Java"
date: 2024-06-18T11:12:01Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

Pour développer des applications multithread robustes, il est indispensable de gérer correctement et efficacement les données partagées. Java propose notamment les variables `volatile` et les classes atomiques du package `java.util.concurrent.atomic`. Ces deux mécanismes permettent à plusieurs threads de manipuler un même état, mais ils répondent à des besoins différents.

**Pourquoi utiliser les classes atomiques ?**

Les classes atomiques permettent d’exécuter des opérations indivisibles sur une variable sans synchronisation explicite. Une incrémentation, une décrémentation, une addition ou une opération compare-and-set (CAS) s’accomplit comme une seule étape logique : aucun autre thread ne peut observer un état intermédiaire ni intervenir entre la lecture et l’écriture.

Le package `java.util.concurrent.atomic` contient notamment :

- `AtomicInteger` ;
- `AtomicLong` ;
- `AtomicBoolean` ;
- `AtomicReference`.

Ces classes proposent des méthodes thread-safe pour lire, modifier et mettre à jour conditionnellement une valeur. Dans les scénarios simples, elles évitent d’utiliser `synchronized` ou un verrou explicite.

**Principales caractéristiques des classes atomiques**

- **Atomicité.** L’opération est indivisible : les autres threads ne voient que l’état avant ou après celle-ci.
- **Exécution non bloquante.** La plupart des méthodes s’appuient sur des instructions bas niveau du processeur et ne suspendent pas un thread dans l’attente d’un moniteur.
- **Approche lock-free.** L’exclusion mutuelle n’est généralement pas utilisée, ce qui réduit les risques de longue contention et d’interblocage.
- **Visibilité.** Les lectures et écritures bénéficient des garanties du Java Memory Model ; le résultat d’une mise à jour est donc observable par les autres threads.

Cela ne signifie pas que les classes atomiques sont toujours plus rapides qu’un verrou. En cas de forte contention, les tentatives répétées de CAS peuvent elles aussi coûter cher. Le choix dépend de l’algorithme et de la charge réelle.

**Exemple avec AtomicInteger**

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicExample {
    private AtomicInteger counter = new AtomicInteger(0);

    public void increment() {
        counter.incrementAndGet();  // incrémentation atomique
    }

    public int getValue() {
        return counter.get();
    }

    public static void main(String[] args) {
        AtomicExample example = new AtomicExample();

        // Création de 1000 threads qui incrémentent le compteur
        Thread[] threads = new Thread[1000];
        for (int i = 0; i < 1000; i++) {
            threads[i] = new Thread(example::increment);
            threads[i].start();
        }

        // Attente de la fin de tous les threads
        for (Thread thread : threads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }

        System.out.println("Final counter value: " + example.getValue());
    }
}
```

`incrementAndGet()` regroupe la lecture, l’incrémentation et l’écriture dans une seule opération atomique. Lorsque les 1000 threads sont terminés, le compteur contient donc la valeur attendue. Un simple `int`, même déclaré `volatile`, pourrait perdre des mises à jour lors d’exécutions concurrentes de `++`.

**Différences entre volatile et les classes atomiques**

1. **Visibilité et atomicité.**
   - `volatile` garantit que les threads voient les écritures récentes, mais ne protège pas les actions composées.
   - Les classes atomiques assurent à la fois visibilité et atomicité pour les opérations qu’elles exposent.
2. **Mises à jour complexes.**
   - `volatile` convient aux lectures et affectations indépendantes.
   - Les classes atomiques proposent `incrementAndGet()`, `decrementAndGet()`, `addAndGet()` et `compareAndSet()` pour sécuriser les opérations read-modify-write.
3. **Implémentation.**
   - `volatile` s’appuie sur les règles du Java Memory Model pour publier les valeurs et ordonner les accès.
   - Les classes atomiques utilisent généralement le CAS matériel et recommencent l’opération si nécessaire.
4. **Cas d’usage.**
   - `volatile` convient aux drapeaux, aux indicateurs de disponibilité et aux états simples.
   - Les classes atomiques sont adaptées aux compteurs, aux références et aux valeurs fréquemment mises à jour par plusieurs threads.

**Résumé**

Comprendre la différence entre `volatile` et les classes atomiques permet de choisir le mécanisme le plus simple qui garantisse la correction. `volatile` résout la visibilité, tandis que les classes atomiques ajoutent des opérations composées indivisibles. Lorsque l’invariant concerne une seule variable, elles offrent souvent une solution non bloquante claire. S’il faut modifier plusieurs valeurs ensemble, exécuter une longue section critique ou attendre une condition, des verrous ou d’autres outils de concurrence de haut niveau sont plus appropriés.
