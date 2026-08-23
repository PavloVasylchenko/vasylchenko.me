---
title: "Comprendre Exchanger en Java"
date: 2024-06-21T07:16:11Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Qu’est-ce que Exchanger en Java ?**

La classe `Exchanger` crée un point de synchronisation où deux threads forment une paire et échangent des objets. Chaque thread transmet sa valeur à `exchange()`, attend son partenaire puis, lorsque les deux se rencontrent, reçoit l’objet fourni par l’autre.

Ce mécanisme est utile lorsque deux threads doivent se transmettre directement des données. Par exemple, un producteur peut remplir un tampon pendant qu’un consommateur traite le précédent, puis ils échangent leurs tampons sans utiliser de file commune. `Exchanger` assure à la fois le transfert de la valeur et la synchronisation du moment de l’échange.

**Principales caractéristiques d’Exchanger**

1. **Échange d’objets.** Deux threads donnent et reçoivent des valeurs du même type générique.
2. **Opération bloquante.** `exchange()` attend qu’un autre thread atteigne le même point.
3. **Délai maximal.** Une surcharge de la méthode limite l’attente et évite un blocage permanent.
4. **Fonctionnement par paires.** Si plus de deux threads arrivent, `Exchanger` les associe par paires ; il n’est pas possible de choisir à l’avance un partenaire précis.

**Utilisation de base d’Exchanger**

```java
import java.util.concurrent.Exchanger;

public class ExchangerExample {
    private static final Exchanger<String> exchanger = new Exchanger<>();

    public static void main(String[] args) {
        new Thread(new Producer()).start();
        new Thread(new Consumer()).start();
    }

    static class Producer implements Runnable {
        @Override
        public void run() {
            try {
                String data = "Data from Producer";
                System.out.println("Producer: " + data);
                String response = exchanger.exchange(data);
                System.out.println("Producer received: " + response);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

    static class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                String data = "Data from Consumer";
                System.out.println("Consumer: " + data);
                String response = exchanger.exchange(data);
                System.out.println("Consumer received: " + response);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

Dans cet exemple :

1. **Initialisation.** Un `Exchanger<String>` est créé ; les participants échangent donc des chaînes.
2. **Producer et Consumer.** Les deux classes implémentent `Runnable`, préparent leurs données et appellent la même instance avec `exchanger.exchange()`.
3. **Exécution.** Le premier thread arrivé se bloque. Lorsque le second atteint le point d’échange, les valeurs sont permutées atomiquement et les deux threads reprennent leur travail.

Si un participant est interrompu pendant l’attente, `exchange()` lève `InterruptedException`. Le code rétablit le statut d’interruption afin que la couche appelante puisse gérer correctement l’annulation.

**Exchanger avec délai maximal**

Si le partenaire risque de ne jamais arriver, une attente illimitée est indésirable. La version temporisée permet de terminer l’opération de manière prévisible.

```java
import java.util.concurrent.Exchanger;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class ExchangerTimeoutExample {
    private static final Exchanger<String> exchanger = new Exchanger<>();

    public static void main(String[] args) {
        new Thread(new Producer()).start();
        new Thread(new Consumer()).start();
    }

    static class Producer implements Runnable {
        @Override
        public void run() {
            try {
                String data = "Data from Producer";
                System.out.println("Producer: " + data);
                String response = exchanger.exchange(data, 5, TimeUnit.SECONDS);
                System.out.println("Producer received: " + response);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } catch (TimeoutException e) {
                System.out.println("Producer timed out");
            }
        }
    }

    static class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                String data = "Data from Consumer";
                System.out.println("Consumer: " + data);
                String response = exchanger.exchange(data, 5, TimeUnit.SECONDS);
                System.out.println("Consumer received: " + response);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } catch (TimeoutException e) {
                System.out.println("Consumer timed out");
            }
        }
    }
}
```

Les deux threads attendent au maximum cinq secondes. Si le partenaire n’arrive pas, une `TimeoutException` est levée et l’application peut réessayer, libérer des ressources ou abandonner la tâche. Il vaut mieux traiter séparément timeout et interruption : le premier peut être un résultat normal du protocole, tandis que la seconde signale généralement une demande d’annulation.

**Quand utiliser Exchanger ?**

- pour échanger un tampon plein contre un tampon vide ;
- pour comparer ou combiner les résultats intermédiaires de deux workers ;
- dans un pipeline où deux étapes doivent se rejoindre après chaque tour ;
- dans les tests concurrents nécessitant un point précis de synchronisation entre deux threads.

Avec plusieurs producteurs et consommateurs, `BlockingQueue` est généralement plus adaptée. `Exchanger` est surtout efficace lorsque le protocole se compose naturellement de paires.

**Résumé**

`Exchanger` réunit transfert de données et synchronisation de deux threads dans une seule opération. Il rend explicite la communication par paires et évite un conteneur partagé séparé. Le délai maximal protège contre l’attente infinie, tandis qu’une bonne gestion des interruptions permet d’annuler le travail. Dans les bons scénarios, cette classe simplifie la coordination et clarifie l’algorithme concurrent.
