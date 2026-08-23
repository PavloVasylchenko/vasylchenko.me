---
title: "Utiliser CyclicBarrier en Java"
date: 2024-06-19T13:15:18Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Comprendre CyclicBarrier en Java**

La classe `CyclicBarrier` est un outil de synchronisation qui permet à un groupe de threads de s’attendre à un point commun. Elle est utile lorsque plusieurs threads exécutent indépendamment une phase de travail, mais ne peuvent commencer la suivante qu’une fois tous arrivés au même état.

Contrairement à `CountDownLatch`, la barrière peut être réutilisée après la libération des threads. `CyclicBarrier` convient donc aux algorithmes cycliques ou itératifs dans lesquels les mêmes participants se synchronisent à la fin de chaque phase.

**Principales caractéristiques de CyclicBarrier**

- **Réutilisable.** Une fois tous les threads arrivés et libérés, un nouveau cycle de la barrière commence.
- **Action facultative.** Le constructeur peut recevoir un `Runnable` exécuté par le dernier thread arrivé avant que les autres ne continuent.
- **Nombre fixe de participants.** La valeur parties est définie à la création et indique combien d’appels à `await()` sont nécessaires pour ouvrir la barrière.

**Utilisation de base de CyclicBarrier**

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierExample {
    private static final int NUMBER_OF_THREADS = 3;
    private static final CyclicBarrier barrier = new CyclicBarrier(NUMBER_OF_THREADS, new Runnable() {
        @Override
        public void run() {
            System.out.println("All parties have arrived at the barrier, lets play");
        }
    });

    public static void main(String[] args) {
        for (int i = 0; i < NUMBER_OF_THREADS; i++) {
            new Thread(new Task()).start();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " is waiting at the barrier.");
                barrier.await();
                System.out.println(Thread.currentThread().getName() + " has crossed the barrier.");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```

Dans cet exemple :

- **Initialisation.** La barrière reçoit `NUMBER_OF_THREADS` et une action facultative lancée à l’arrivée du dernier participant.
- **Tâche.** Chaque `Task` appelle `barrier.await()` et reste suspendu jusqu’à l’arrivée des autres.
- **Exécution.** Trois threads démarrent. Après le troisième appel à `await()`, l’action de la barrière s’exécute, puis tous les participants poursuivent leur travail.

**Utilisation avancée de CyclicBarrier**

1. **Barrière cassée.** Si un thread en attente est interrompu ou dépasse son délai, la barrière passe à l’état broken. Les autres participants reçoivent `BrokenBarrierException`. Cet état se vérifie avec `barrier.isBroken()`.
2. **Délai maximal.** Une surcharge de `await()` évite d’attendre indéfiniment :

   ```java
   barrier.await(5, TimeUnit.SECONDS);
   ```
3. **Réinitialisation.** `barrier.reset()` remet l’objet dans son état initial. Les threads déjà en attente reçoivent une exception ; le reset doit donc faire partie d’un protocole coordonné.

**Exemple avec délai et réinitialisation**

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class CyclicBarrierAdvancedExample {
    private static final int NUMBER_OF_THREADS = 3;
    private static final CyclicBarrier barrier = new CyclicBarrier(NUMBER_OF_THREADS, new Runnable() {
        @Override
        public void run() {
            System.out.println("All parties have arrived at the barrier, lets play");
        }
    });

    public static void main(String[] args) {
        for (int i = 0; i < NUMBER_OF_THREADS; i++) {
            new Thread(new Task()).start();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " is waiting at the barrier.");
                barrier.await(5, TimeUnit.SECONDS);
                System.out.println(Thread.currentThread().getName() + " has crossed the barrier.");
            } catch (InterruptedException | BrokenBarrierException | TimeoutException e) {
                e.printStackTrace();
                if (barrier.isBroken()) {
                    System.out.println("Barrier is broken, resetting...");
                    barrier.reset();
                }
            }
        }
    }
}
```

Chaque thread attend au maximum cinq secondes. Si tout le monde arrive à temps, l’action s’exécute et la phase suivante peut commencer. Si le délai expire, une `TimeoutException` est levée et la barrière devient cassée. L’exemple détecte cet état puis réinitialise l’objet pour un usage ultérieur.

Dans un système réel, il faut éviter que plusieurs threads appellent `reset()` sans coordination. Un composant superviseur décide généralement si la phase peut être rejouée en toute sécurité et pilote la récupération.

**Résumé**

`CyclicBarrier` synchronise un nombre fixe de threads à un point commun. Sa réutilisation et son action facultative la rendent pratique pour les calculs parallèles organisés en plusieurs tours. Les délais et la gestion de l’état broken empêchent un blocage définitif lorsqu’un participant échoue. Avec un protocole clair, `CyclicBarrier` permet de coordonner de façon robuste des tâches cycliques.
