---
title: "Comprendre ReentrantLock en Java"
date: 2024-06-18T14:11:55Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

En Java, il est essentiel de contrôler correctement l’accès concurrent aux ressources partagées afin de préserver l’intégrité des données et d’éviter les conditions de concurrence. Le package `java.util.concurrent.locks` fournit plusieurs mécanismes de verrouillage, dont le très courant `ReentrantLock`.

**Qu’est-ce que ReentrantLock ?**

`ReentrantLock` est un outil de synchronisation dont l’API est plus flexible qu’un bloc `synchronized`. Il expose des opérations explicites d’acquisition et de libération. « Réentrant » signifie que le thread propriétaire peut reprendre le même verrou sans se bloquer lui-même. Un compteur interne augmente à chaque `lock()` et diminue à chaque `unlock()`.

**Fonctionnalités principales de ReentrantLock**

1. **Réentrance.** Le même thread peut acquérir plusieurs fois le verrou. Il doit effectuer autant d’appels à `unlock()` pour le libérer complètement.
2. **Équité.** Le verrou peut être équitable ou non. Le mode équitable privilégie généralement le thread qui attend depuis le plus longtemps ; le mode non équitable autorise un nouveau thread à passer devant.
3. **Condition.** `newCondition()` crée des conditions permettant aux threads d’attendre un état précis et d’être signalés lorsqu’il change.
4. **Attente interruptible.** `lockInterruptibly()` permet d’interrompre un thread en attente du verrou.
5. **Tentative sans attente.** `tryLock()` renvoie immédiatement un résultat ou attend au maximum pendant le délai fourni.

**Utilisation de base de ReentrantLock**

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockExample {
    private final Lock lock = new ReentrantLock();
    private int counter = 0;

    public void increment() {
        lock.lock();
        try {
            counter++;
        } finally {
            lock.unlock();
        }
    }

    public int getCounter() {
        lock.lock();
        try {
            return counter;
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReentrantLockExample example = new ReentrantLockExample();

        Thread[] threads = new Thread[1000];
        for (int i = 0; i < 1000; i++) {
            threads[i] = new Thread(example::increment);
            threads[i].start();
        }

        for (Thread thread : threads) {
            thread.join();
        }

        System.out.println("Final counter value: " + example.getCounter());
    }
}
```

Ici, `ReentrantLock` protège `increment()` et `getCounter()` contre les accès simultanés. `lock()` acquiert le verrou et `unlock()` est toujours placé dans `finally`, afin de libérer le lock même si la section critique lève une exception. Aucun code susceptible d’échouer ne doit être exécuté entre `lock()` et le début du `try`.

**Fonctionnalités avancées**

1. **Verrou équitable.** Passez `true` au constructeur :

   ```java
   Lock fairLock = new ReentrantLock(true);
   ```

   Cette option réduit le risque de famine pour les threads en attente depuis longtemps, mais peut diminuer le débit global.
2. **Try Lock.** Une tentative non bloquante permet de choisir une autre stratégie :

   ```java
   if (lock.tryLock()) {
       try {
           // Opérations protégées par le verrou
       } finally {
           lock.unlock();
       }
   } else {
       // Le verrou n'a pas pu être acquis
   }
   ```
3. **Conditions.** Elles servent à mettre en place une coordination plus complexe :

   ```java
   Lock lock = new ReentrantLock();
   Condition condition = lock.newCondition();

   public void awaitCondition() throws InterruptedException {
       lock.lock();
       try {
           condition.await();
       } finally {
           lock.unlock();
       }
   }

   public void signalCondition() {
       lock.lock();
       try {
           condition.signal();
       } finally {
           lock.unlock();
       }
   }
   ```

`await()` libère atomiquement le verrou et suspend le thread. Après le signal, celui-ci doit reprendre le verrou avant de continuer. En pratique, la condition est vérifiée dans une boucle pour gérer les réveils intempestifs et les modifications effectuées par d’autres threads.

**Quand préférer ReentrantLock à synchronized ?**

1. Lorsqu’un ordre équitable est nécessaire pour limiter la famine.
2. Lorsque les threads en attente doivent pouvoir être interrompus.
3. Lorsqu’il faut tenter une acquisition sans blocage ou avec un délai maximal.
4. Lorsque plusieurs files `Condition` sont requises pour un protocole complexe d’attente et de notification.

**Résumé**

`ReentrantLock` est un mécanisme de synchronisation puissant et flexible. Il complète `synchronized` avec l’équité, l’acquisition interruptible, `tryLock()` et les variables de condition. Cette souplesse impose une gestion explicite : chaque acquisition réussie doit être suivie d’une libération dans `finally`. Correctement utilisé, il facilite la création d’un code concurrent robuste et maintenable.
