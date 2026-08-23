---
title: "CountDownLatch en Java avec des exemples"
date: 2024-06-22T08:57:01Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**CountDownLatch en Java avec des exemples**

Dans les applications Java concurrentes, il est fréquent de devoir synchroniser plusieurs threads. La classe `CountDownLatch` simplifie cette coordination : elle permet à un ou plusieurs threads d’attendre la fin d’un ensemble d’opérations exécutées par d’autres threads.

**Qu’est-ce que CountDownLatch ?**

Imaginez le départ d’une course. Tous les coureurs doivent partir ensemble et attendent donc la fin d’un compte à rebours. Lorsque le compteur atteint zéro, la course commence. `CountDownLatch` fonctionne de la même manière : il est créé avec un nombre d’événements ou de tâches qui doivent se produire avant que les threads en attente puissent continuer.

Le latch est à usage unique. Une fois arrivé à zéro, son compteur ne peut pas être incrémenté ni réinitialisé. Si le même groupe de threads doit réutiliser une barrière, `CyclicBarrier` est généralement plus appropriée.

**Comment fonctionne CountDownLatch ?**

1. **Initialisation.** L’objet reçoit un count initial, c’est-à-dire le nombre d’appels à `countDown()` nécessaires pour ouvrir le latch.
2. **Décompte.** Chaque appel à `countDown()` réduit la valeur d’une unité. Les appels supplémentaires après zéro n’ont aucun effet.
3. **Attente.** `await()` bloque le thread appelant jusqu’à ce que le compteur atteigne zéro.
4. **Visibilité.** Les actions effectuées avant `countDown()` sont visibles par le thread qui revient correctement de `await()`.

**Exemple de base**

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {
    private static final int NUMBER_OF_TASKS = 3;
    private static final CountDownLatch latch = new CountDownLatch(NUMBER_OF_TASKS);

    public static void main(String[] args) {
        for (int i = 0; i < NUMBER_OF_TASKS; i++) {
            new Thread(new Task()).start();
        }

        try {
            latch.await();  // Le thread principal attend que le compteur soit à zéro
            System.out.println("All tasks are completed. Main thread resumes.");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                // Simulation d'un traitement
                Thread.sleep(1000);
                System.out.println(Thread.currentThread().getName() + " completed.");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                latch.countDown();  // Décrémentation dans tous les cas
            }
        }
    }
}
```

Le `CountDownLatch` est initialisé à 3. Trois tâches sont lancées ; chacune simule une seconde de travail, puis appelle `countDown()`. Le thread principal s’arrête sur `latch.await()` et ne reprend qu’une fois les trois tâches terminées.

L’appel se trouve dans `finally`, afin que l’échec d’une tâche ne laisse pas le thread principal attendre indéfiniment. Cette conception signifie « la tâche a terminé sa tentative », et non nécessairement « la tâche a réussi » ; une application réelle doit stocker séparément les erreurs et les résultats.

**Exemple avancé avec délai maximal**

Il n’est pas toujours acceptable d’attendre sans limite. La surcharge temporisée de `await()` renvoie un `boolean` et permet de fixer une durée maximale.

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

public class CountDownLatchTimeoutExample {
    private static final int NUMBER_OF_TASKS = 3;
    private static final CountDownLatch latch = new CountDownLatch(NUMBER_OF_TASKS);

    public static void main(String[] args) {
        for (int i = 0; i < NUMBER_OF_TASKS; i++) {
            new Thread(new Task()).start();
        }

        try {
            boolean completed = latch.await(5, TimeUnit.SECONDS);
            if (completed) {
                System.out.println("All tasks are completed. Main thread resumes.");
            } else {
                System.out.println("Timeout occurred before all tasks completed.");
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println(Thread.currentThread().getName() + " completed.");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                latch.countDown();
            }
        }
    }
}
```

Le thread principal attend au maximum cinq secondes. Si les trois tâches ont décrémenté le compteur pendant ce délai, `await()` renvoie `true`. Sinon, il renvoie `false` et l’application peut annuler les opérations restantes, signaler une erreur ou choisir une solution de repli. Le timeout n’arrête pas automatiquement les worker threads.

**Cas d’usage concrets de CountDownLatch**

1. **Démarrer plusieurs threads en même temps.** Un latch initialisé à 1 maintient tous les workers en attente d’un signal commun :

   ```java
   import java.util.concurrent.CountDownLatch;

   public class ConcurrentStartExample {
       private static final int NUMBER_OF_THREADS = 5;
       private static final CountDownLatch startLatch = new CountDownLatch(1);

       public static void main(String[] args) {
           for (int i = 0; i < NUMBER_OF_THREADS; i++) {
               new Thread(new Worker()).start();
           }

           try {
               System.out.println("Ready... Set...");
               Thread.sleep(1000); // Travail de préparation
               startLatch.countDown();
               System.out.println("Go!");
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
           }
       }

       static class Worker implements Runnable {
           @Override
           public void run() {
               try {
                   startLatch.await();
                   System.out.println(Thread.currentThread().getName() + " is working.");
               } catch (InterruptedException e) {
                   Thread.currentThread().interrupt();
               }
           }
       }
   }
   ```

   Les cinq workers démarrent et attendent. Un seul `countDown()` ouvre le latch et leur permet de poursuivre presque simultanément. Ce modèle est pratique pour les tests de charge ou le lancement d’un groupe de tâches après une phase d’initialisation.

2. **Attendre la fin de plusieurs threads.** Un latch dont la valeur correspond au nombre de tâches permet d’agréger ensuite leurs résultats :

   ```java
   import java.util.concurrent.CountDownLatch;

   public class WaitForCompletionExample {
       private static final int NUMBER_OF_TASKS = 3;
       private static final CountDownLatch completionLatch = new CountDownLatch(NUMBER_OF_TASKS);

       public static void main(String[] args) {
           for (int i = 0; i < NUMBER_OF_TASKS; i++) {
               new Thread(new SubTask()).start();
           }

           try {
               completionLatch.await();
               System.out.println("All subtasks completed. Aggregating results.");
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
           }
       }

       static class SubTask implements Runnable {
           @Override
           public void run() {
               try {
                   Thread.sleep(1000);
                   System.out.println(Thread.currentThread().getName() + " subtask completed.");
               } catch (InterruptedException e) {
                   Thread.currentThread().interrupt();
               } finally {
                   completionLatch.countDown();
               }
           }
       }
   }
   ```

   Le thread principal ne commence l’agrégation qu’après la fin de la tentative de chaque sous-tâche. Dans du code moderne, `ExecutorService`, `Future` ou `CompletableFuture` peuvent résoudre des problèmes similaires, mais `CountDownLatch` reste une façon simple d’exprimer une dépendance envers un nombre fixe d’événements.

**Résumé**

`CountDownLatch` est un mécanisme de coordination robuste et à usage unique. Il convient à un départ commun, à l’attente d’un ensemble de tâches et à la division d’un travail en parties indépendantes. Il faut choisir correctement la valeur initiale, garantir un `countDown()` pour chaque événement attendu et traiter les interruptions. Si l’attente ne doit pas être infinie, utilisez un délai maximal et définissez explicitement le sort des tâches encore actives.
