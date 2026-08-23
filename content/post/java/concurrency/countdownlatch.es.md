---
title: "CountDownLatch en Java con ejemplos"
date: 2024-06-22T08:57:01Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**CountDownLatch en Java con ejemplos**

Al desarrollar aplicaciones concurrentes en Java, es habitual tener que sincronizar varios hilos. La clase `CountDownLatch` facilita esta coordinación: permite que uno o más hilos esperen hasta que termine un conjunto de operaciones ejecutadas por otros hilos.

**¿Qué es CountDownLatch?**

Imagina la salida de una carrera. Todos los corredores deben empezar a la vez, así que esperan a que finalice una cuenta atrás. Cuando el contador llega a cero, comienza la carrera. `CountDownLatch` funciona de manera parecida: se crea con un número de eventos o tareas que deben ocurrir antes de que los hilos en espera puedan continuar.

El latch es de un solo uso. Después de llegar a cero, el contador no puede incrementarse ni volver al valor inicial. Si se necesita una barrera reutilizable para el mismo grupo de hilos, `CyclicBarrier` suele ser una opción más adecuada.

**¿Cómo funciona CountDownLatch?**

1. **Inicialización.** El objeto recibe un count inicial: el número de llamadas a `countDown()` necesarias para abrir el latch.
2. **Cuenta atrás.** Cada `countDown()` reduce el valor en uno. Las llamadas adicionales después de cero no producen ningún efecto.
3. **Espera.** `await()` bloquea al hilo que lo invoca hasta que el contador alcanza cero.
4. **Visibilidad.** Las acciones realizadas antes de `countDown()` son visibles para el hilo que regresa correctamente de `await()`.

**Ejemplo básico**

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
            latch.await();  // El hilo principal espera hasta que el contador sea cero
            System.out.println("All tasks are completed. Main thread resumes.");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                // Simulamos algo de trabajo
                Thread.sleep(1000);
                System.out.println(Thread.currentThread().getName() + " completed.");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                latch.countDown();  // Reducimos el contador en cualquier caso
            }
        }
    }
}
```

El `CountDownLatch` se inicializa con un valor de 3. Se lanzan tres tareas; cada una simula trabajo durante un segundo y luego invoca `countDown()`. El hilo principal se detiene en `latch.await()` y solo continúa cuando las tres tareas han terminado.

La llamada se encuentra en `finally`, de modo que el fallo de una tarea no deja al hilo principal esperando para siempre. Este diseño indica que «la tarea terminó su intento», no necesariamente que finalizó correctamente; una aplicación real debe guardar los errores y los resultados por separado.

**Ejemplo avanzado con timeout**

En ocasiones no se puede esperar indefinidamente. La sobrecarga temporizada de `await()` devuelve un `boolean` y permite establecer un límite.

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

El hilo principal espera como máximo cinco segundos. Si las tres tareas reducen el contador dentro de ese periodo, `await()` devuelve `true`. De lo contrario devuelve `false`, y la aplicación puede cancelar las operaciones pendientes, informar de un error o activar una ruta alternativa. El timeout no detiene automáticamente a los worker threads.

**Casos de uso reales de CountDownLatch**

1. **Iniciar varios hilos al mismo tiempo.** Un latch con valor 1 mantiene a todos los trabajadores en espera hasta recibir una señal común:

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
               Thread.sleep(1000); // Trabajo de preparación
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

   Los cinco trabajadores arrancan y quedan esperando. Una única llamada a `countDown()` abre el latch y les permite continuar casi al mismo tiempo. Es un patrón útil en pruebas de carga o para iniciar tareas después de completar la inicialización.

2. **Esperar a que finalicen varios hilos.** Un latch cuyo valor coincide con el número de tareas permite agregar sus resultados más tarde:

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

   El hilo principal comienza la agregación únicamente después de que todas las subtareas terminan su intento. En código moderno, `ExecutorService`, `Future` o `CompletableFuture` también pueden resolver situaciones similares, pero `CountDownLatch` sigue siendo una forma sencilla de expresar la dependencia respecto a un número fijo de eventos.

**Resumen**

`CountDownLatch` es un mecanismo robusto y de un solo uso para coordinar hilos. Sirve para una salida común, para esperar a un conjunto de tareas y para dividir un trabajo grande en partes independientes. Es importante elegir bien el contador inicial, garantizar un `countDown()` por cada evento esperado y tratar las interrupciones. Si la espera no puede ser infinita, utiliza un timeout y define explícitamente qué ocurrirá con las tareas que sigan ejecutándose.
