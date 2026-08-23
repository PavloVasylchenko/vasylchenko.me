---
title: "Utiliser Semaphore en Java"
date: 2024-06-20T08:16:17Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Qu’est-ce que Semaphore en Java ?**

`Semaphore` est un outil de synchronisation qui limite le nombre de threads pouvant accéder simultanément à une ressource partagée. Il appartient au package `java.util.concurrent` et convient particulièrement aux ressources de capacité fixe : pools de connexions, sockets réseau, imprimantes ou descripteurs de fichiers.

**Principales caractéristiques de Semaphore**

1. **Permis.** Le sémaphore maintient un ensemble de permits. Un thread en acquiert un avant de travailler, puis le restitue lorsqu’il a terminé.
2. **Acquisition bloquante ou non bloquante.** `acquire()` attend un permis disponible, tandis que `tryAcquire()` permet de choisir immédiatement une autre stratégie.
3. **Équité.** Un sémaphore peut être équitable ou non. Le mode équitable attribue généralement les permis dans l’ordre des demandes et réduit le risque de famine.
4. **Plusieurs permis.** Un thread peut acquérir et libérer plusieurs permits si son opération consomme plusieurs unités de la ressource.

**Utilisation de base de Semaphore**

```java
import java.util.concurrent.Semaphore;

public class SemaphoreExample {
   private static final int MAX_PERMITS = 3;
   private static final Semaphore semaphore = new Semaphore(MAX_PERMITS);

   public static void main(String[] args) {
      for (int i = 0; i < 10; i++) {
         new Thread(new Task()).start();
      }
   }

   static class Task implements Runnable {
      @Override
      public void run() {
         boolean acquired = false;
         try {
            System.out.println(Thread.currentThread().getName() + " is waiting for a permit.");
            semaphore.acquire();
            acquired = true;
            System.out.println(Thread.currentThread().getName() + " acquired a permit.");

            // Simulation d'un travail avec la ressource partagée
            Thread.sleep(2000);

            System.out.println(Thread.currentThread().getName() + " releasing a permit.");
         } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
         } finally {
            if (acquired) {
               semaphore.release();
            }
         }
      }
   }
}
```

Le sémaphore est créé avec trois permis. Dix threads sont démarrés, mais seuls trois peuvent franchir `acquire()` et exécuter simultanément la section critique. Les autres attendent. Après un appel à `release()`, le permis redevient disponible pour un autre participant.

La variable `acquired` est importante : si le thread a été interrompu avant l’acquisition, il ne doit pas appeler `release()`, sous peine d’augmenter par erreur le nombre total de permis.

**Utilisation avancée de Semaphore**

1. **Tentative non bloquante.** Le thread vérifie la disponibilité et choisit immédiatement une solution de repli :

   ```java
   if (semaphore.tryAcquire()) {
       try {
           // Section critique
       } finally {
           semaphore.release();
       }
   } else {
       System.out.println(Thread.currentThread().getName() + " could not acquire a permit.");
   }
   ```
2. **Acquisition avec délai.** Une attente limitée évite de rester bloqué en période de surcharge :

   ```java
   if (semaphore.tryAcquire(1, TimeUnit.SECONDS)) {
       try {
           // Section critique
       } finally {
           semaphore.release();
       }
   } else {
       System.out.println(Thread.currentThread().getName() + " timed out waiting for a permit.");
   }
   ```
3. **Sémaphore équitable.** Le second argument du constructeur active l’ordre d’arrivée :

   ```java
   Semaphore fairSemaphore = new Semaphore(MAX_PERMITS, true);
   ```

L’équité est utile lorsqu’aucun thread ne doit attendre trop longtemps, mais la gestion de la file peut réduire le débit global.

**Cas d’usage**

1. **Limitation du parallélisme.** Contrôler le nombre de requêtes simultanées vers un service externe ou une opération coûteuse.
2. **Pools de connexions.** Chaque permis représente une connexion disponible à la base de données.
3. **Gestion de ressources.** Restreindre l’accès à quelques imprimantes, fichiers ou périphériques.
4. **Protection contre la surcharge.** Les tâches supplémentaires attendent ou sont rejetées au lieu de consommer ensemble mémoire et CPU.

**Résumé**

`Semaphore` offre un mécanisme robuste pour limiter l’accès concurrent aux ressources partagées. Contrairement à une exclusion mutuelle classique, il peut laisser avancer un nombre défini de threads en parallèle. Il faut libérer uniquement les permis réellement acquis et toujours le faire dans `finally`. Bien employé, un sémaphore aide à exploiter efficacement des ressources limitées et à réduire la contention.
