---
title: "Colecciones concurrentes en Java con ejemplos"
date: 2024-06-25T13:46:57Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Colecciones concurrentes en Java con ejemplos**

El paquete `java.util.concurrent` incluye varias colecciones diseñadas para entornos multihilo. Proporcionan operaciones thread-safe y suelen escalar mejor que una colección convencional protegida por un único bloque `synchronized`. En este artículo veremos las implementaciones principales, sus casos de uso y ejemplos prácticos.

**Introducción a las colecciones concurrentes**

Java ofrece, entre otras, estas colecciones:

- `ConcurrentHashMap`;
- `CopyOnWriteArrayList`;
- `CopyOnWriteArraySet`;
- `ConcurrentLinkedQueue`;
- `ConcurrentLinkedDeque`;
- `LinkedBlockingQueue`;
- `PriorityBlockingQueue`.

Cada estructura aplica un modelo de concurrencia distinto. Que un método sea seguro para hilos no vuelve atómica cualquier secuencia de llamadas; para las acciones compuestas hay que utilizar métodos atómicos de la colección o sincronización adicional.

**ConcurrentHashMap**

`ConcurrentHashMap` es una implementación thread-safe de map que admite lecturas y actualizaciones concurrentes. No bloquea toda la tabla en cada operación y ofrece métodos atómicos como `putIfAbsent`, `compute`, `merge` y `replace`.

```java
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapExample {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

        Runnable writerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                map.put(Thread.currentThread().getName() + i, i);
            }
        };

        Thread writer1 = new Thread(writerTask);
        Thread writer2 = new Thread(writerTask);
        writer1.start();
        writer2.start();

        try {
            writer1.join();
            writer2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        Runnable readerTask = () -> map.forEach((key, value) ->
                System.out.println(Thread.currentThread().getName() + ": " + key + ": " + value));

        new Thread(readerTask).start();
        new Thread(readerTask).start();
    }
}
```

Dos writers añaden elementos al mismo tiempo y después dos readers recorren el mapa. La iteración es weakly consistent: no lanza `ConcurrentModificationException` y puede observar parte de los cambios que suceden en paralelo.

**CopyOnWriteArrayList**

`CopyOnWriteArrayList` es una variante thread-safe de `ArrayList`. Cada operación de escritura crea una copia nueva del array interno. Las lecturas y recorridos son rápidos y no bloquean, pero escribir con frecuencia implica un coste elevado de memoria y copias.

```java
import java.util.concurrent.CopyOnWriteArrayList;

public class CopyOnWriteArrayListExample {
    public static void main(String[] args) {
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

        Runnable writerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                list.add(Thread.currentThread().getName() + i);
            }
        };

        Thread writer1 = new Thread(writerTask);
        Thread writer2 = new Thread(writerTask);
        writer1.start();
        writer2.start();

        try {
            writer1.join();
            writer2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        Runnable readerTask = () -> {
            for (String item : list) {
                System.out.println(Thread.currentThread().getName() + ": " + item);
            }
        };

        new Thread(readerTask).start();
        new Thread(readerTask).start();
    }
}
```

El iterador trabaja sobre una instantánea del array tomada al crearse y no ve modificaciones posteriores. Esta colección es apropiada para listas de suscriptores o configuraciones que se leen a menudo y cambian muy pocas veces.

**CopyOnWriteArraySet**

`CopyOnWriteArraySet` es un conjunto thread-safe implementado mediante `CopyOnWriteArrayList`. Mantiene elementos únicos y hereda el mismo modelo: lecturas estables y rápidas, iteradores snapshot y modificaciones costosas.

```java
import java.util.concurrent.CopyOnWriteArraySet;

public class CopyOnWriteArraySetExample {
    public static void main(String[] args) {
        CopyOnWriteArraySet<String> set = new CopyOnWriteArraySet<>();

        Runnable writerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                set.add(Thread.currentThread().getName() + i);
            }
        };

        Thread writer1 = new Thread(writerTask);
        Thread writer2 = new Thread(writerTask);
        writer1.start();
        writer2.start();

        try {
            writer1.join();
            writer2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        Runnable readerTask = () -> {
            for (String item : set) {
                System.out.println(Thread.currentThread().getName() + ": " + item);
            }
        };

        new Thread(readerTask).start();
        new Thread(readerTask).start();
    }
}
```

Es una buena opción para conjuntos pequeños de listeners o permisos donde los recorridos son mucho más frecuentes que las altas y bajas.

**ConcurrentLinkedQueue**

`ConcurrentLinkedQueue` es una cola FIFO no acotada y thread-safe basada en nodos enlazados. Utiliza un algoritmo no bloqueante y escala bien cuando muchos hilos añaden y extraen elementos simultáneamente.

```java
import java.util.concurrent.ConcurrentLinkedQueue;

public class ConcurrentLinkedQueueExample {
    public static void main(String[] args) {
        ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();

        Runnable producerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                queue.add(Thread.currentThread().getName() + i);
            }
        };

        Thread producer1 = new Thread(producerTask);
        Thread producer2 = new Thread(producerTask);
        producer1.start();
        producer2.start();

        try {
            producer1.join();
            producer2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        Runnable consumerTask = () -> {
            String item;
            while ((item = queue.poll()) != null) {
                System.out.println(Thread.currentThread().getName() + ": " + item);
            }
        };

        new Thread(consumerTask).start();
        new Thread(consumerTask).start();
    }
}
```

`poll()` extrae la cabeza de forma atómica o devuelve `null`. No es necesario comprobar `isEmpty()` antes: el estado podría cambiar entre ambas llamadas. La cola no permite esperar de forma bloqueante ni limita a los productores, por lo que el backpressure debe resolverse por separado.

**ConcurrentLinkedDeque**

`ConcurrentLinkedDeque` es una cola doble no bloqueante y thread-safe. Los hilos pueden añadir y retirar elementos de ambos extremos de forma concurrente.

```java
import java.util.concurrent.ConcurrentLinkedDeque;

public class ConcurrentLinkedDequeExample {
    public static void main(String[] args) {
        ConcurrentLinkedDeque<String> deque = new ConcurrentLinkedDeque<>();

        Runnable producerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                deque.addLast(Thread.currentThread().getName() + i);
            }
        };

        Thread producer1 = new Thread(producerTask);
        Thread producer2 = new Thread(producerTask);
        producer1.start();
        producer2.start();

        try {
            producer1.join();
            producer2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        Runnable consumerTask = () -> {
            String item;
            while ((item = deque.pollFirst()) != null) {
                System.out.println(Thread.currentThread().getName() + ": " + item);
            }
        };

        new Thread(consumerTask).start();
        new Thread(consumerTask).start();
    }
}
```

El acceso por ambos lados resulta útil en work stealing, historiales de tareas y algoritmos que combinan operaciones FIFO y LIFO.

**LinkedBlockingQueue**

`LinkedBlockingQueue` es una cola FIFO bloqueante, opcionalmente acotada y basada en nodos enlazados. `take()` espera hasta que aparece un elemento y `put()` espera espacio libre en una cola limitada. Es una base natural para producer-consumer y backpressure sencillo.

```java
import java.util.concurrent.LinkedBlockingQueue;

public class LinkedBlockingQueueExample {
    public static void main(String[] args) {
        LinkedBlockingQueue<String> queue = new LinkedBlockingQueue<>(10);

        Runnable producerTask = () -> {
            try {
                for (int i = 0; i < 1000; i++) {
                    queue.put(Thread.currentThread().getName() + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        Runnable consumerTask = () -> {
            try {
                while (!Thread.currentThread().isInterrupted()) {
                    String item = queue.take();
                    System.out.println(Thread.currentThread().getName() + ": " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        new Thread(producerTask).start();
        new Thread(producerTask).start();
        new Thread(consumerTask).start();
        new Thread(consumerTask).start();
    }
}
```

El límite de 10 obliga a los producers rápidos a esperar cuando los consumers no dan abasto. Una aplicación real necesita un protocolo de finalización: interrupción, un poison pill especial o gestión mediante un executor.

**PriorityBlockingQueue**

`PriorityBlockingQueue` es una cola bloqueante no acotada que utiliza las reglas de orden de `PriorityQueue`. `take()` espera a que exista un elemento, pero los productores nunca se bloquean por capacidad. Los elementos deben implementar `Comparable` o la cola debe recibir un `Comparator`.

```java
import java.util.concurrent.PriorityBlockingQueue;

public class PriorityBlockingQueueExample {
    public static void main(String[] args) {
        PriorityBlockingQueue<Integer> queue = new PriorityBlockingQueue<>();

        Runnable producerTask = () -> {
            for (int i = 0; i < 1000; i++) {
                queue.put((int) (Math.random() * 1000));
            }
        };

        Runnable consumerTask = () -> {
            try {
                while (!Thread.currentThread().isInterrupted()) {
                    Integer item = queue.take();
                    System.out.println(Thread.currentThread().getName() + ": " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        new Thread(producerTask).start();
        new Thread(producerTask).start();
        new Thread(consumerTask).start();
        new Thread(consumerTask).start();
    }
}
```

La cola respeta la prioridad entre los elementos disponibles al extraer, pero el orden global observado depende de las inserciones y extracciones concurrentes. Como la capacidad no tiene límite, la aplicación debe controlar por separado el crecimiento de memoria.

**Tabla comparativa**

| Colección | Thread-safe | Operaciones bloqueantes | Caso de uso |
|---|---|---|---|
| `ConcurrentHashMap` | Sí | No | Operaciones frecuentes con pares clave-valor |
| `CopyOnWriteArrayList` | Sí | No | Muchas lecturas y pocas escrituras |
| `CopyOnWriteArraySet` | Sí | No | Conjunto pequeño recorrido con frecuencia |
| `ConcurrentLinkedQueue` | Sí | No | Operaciones FIFO no bloqueantes y escalables |
| `ConcurrentLinkedDeque` | Sí | No | Acceso concurrente por ambos extremos |
| `LinkedBlockingQueue` | Sí | Sí | FIFO acotada para producer-consumer |
| `PriorityBlockingQueue` | Sí | Sí, al leer | Consumo según prioridad |

**Resumen**

Las colecciones concurrentes permiten manipular datos desde varios hilos sin bloquear manualmente cada operación. La elección correcta depende tanto de la interfaz como del patrón de carga. `ConcurrentHashMap` escala bien con lecturas y actualizaciones frecuentes. Las estructuras copy-on-write sirven para conjuntos casi inmutables con muchos lectores. Las linked queues ofrecen intercambio no bloqueante, mientras que las blocking queues coordinan productores y consumidores y pueden aportar backpressure.

Hay que considerar la semántica de los iteradores, la capacidad, el coste de copiar y el mecanismo de cierre de los consumidores. Una colección thread-safe tampoco protege un invariante de negocio que abarque varias llamadas. Al escoger la estructura adecuada y utilizar sus métodos atómicos se reducen las race conditions, los deadlocks y la contención innecesaria, obteniendo código más escalable y mantenible.
