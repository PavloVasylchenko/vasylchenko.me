---
title: "Uso de CyclicBarrier en Java"
date: 2024-06-19T13:15:18Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Cómo funciona CyclicBarrier en Java**

La clase `CyclicBarrier` es una ayuda de sincronización que permite a un grupo de hilos esperarse en un punto común. Resulta útil cuando varios hilos ejecutan una fase de forma independiente, pero ninguno puede continuar con la siguiente hasta que todos hayan alcanzado el mismo estado.

A diferencia de `CountDownLatch`, la barrera puede reutilizarse después de liberar a los hilos. Por eso `CyclicBarrier` encaja bien en algoritmos cíclicos o iterativos donde los mismos participantes se sincronizan al terminar cada fase.

**Características principales de CyclicBarrier**

- **Reutilizable.** Cuando todos los hilos llegan y son liberados, comienza un nuevo ciclo de la barrera.
- **Acción opcional.** El constructor puede recibir un `Runnable` que ejecutará el último hilo en llegar antes de que los demás continúen.
- **Número fijo de participantes.** El valor parties se define al crear la barrera y determina cuántas llamadas a `await()` hacen falta para abrirla.

**Ejemplo básico de CyclicBarrier**

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

En este ejemplo:

- **Inicialización.** La barrera recibe `NUMBER_OF_THREADS` y una acción opcional que se ejecuta cuando llega el último participante.
- **Implementación de la tarea.** Cada `Task` invoca `barrier.await()` y queda detenido hasta que llegan los demás.
- **Ejecución.** Se inician tres hilos. Tras la tercera llamada a `await()`, se ejecuta la acción de la barrera y todos pueden continuar.

**Uso avanzado de CyclicBarrier**

1. **Barrera rota.** Si uno de los hilos es interrumpido o supera el tiempo de espera, la barrera pasa al estado broken. Los demás participantes reciben `BrokenBarrierException`. Puede consultarse con `barrier.isBroken()`.
2. **Tiempo límite.** La sobrecarga de `await()` evita esperas indefinidas:

   ```java
   barrier.await(5, TimeUnit.SECONDS);
   ```
3. **Reinicio.** `barrier.reset()` devuelve el objeto a su estado inicial. Los hilos que estuviesen esperando recibirán una excepción, de modo que el reinicio debe formar parte de un protocolo controlado.

**Ejemplo con timeout y reinicio**

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

Cada hilo espera como máximo cinco segundos. Si todos llegan a tiempo, se ejecuta la acción y comienza la siguiente fase. Si se supera el límite, se lanza `TimeoutException` y la barrera queda rota. El ejemplo comprueba ese estado y reinicia el objeto para permitir un nuevo uso.

En una aplicación real conviene evitar llamadas simultáneas y descoordinadas a `reset()`. Normalmente un componente supervisor decide si la fase puede repetirse de forma segura y se encarga de recuperar la barrera.

**Resumen**

`CyclicBarrier` sincroniza un número fijo de hilos en un punto común. Su reutilización y la acción opcional la hacen apropiada para cálculos paralelos divididos en rondas. Los timeouts y el manejo del estado broken evitan que el sistema quede bloqueado cuando falla un participante. Con un protocolo bien definido, `CyclicBarrier` permite coordinar tareas cíclicas de manera robusta.
