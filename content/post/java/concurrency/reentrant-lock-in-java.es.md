---
title: "Cómo funciona ReentrantLock en Java"
date: 2024-06-18T14:11:55Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

En Java es fundamental controlar correctamente el acceso concurrente a los recursos compartidos para mantener la integridad de los datos y evitar condiciones de carrera. El paquete `java.util.concurrent.locks` ofrece varios mecanismos de bloqueo; uno de los más utilizados es `ReentrantLock`.

**¿Qué es ReentrantLock?**

`ReentrantLock` es un primitivo de sincronización con una API más flexible que un bloque `synchronized`. Proporciona operaciones explícitas para adquirir y liberar el bloqueo. «Reentrant» significa que el hilo propietario puede adquirir el mismo lock otra vez sin bloquearse a sí mismo. Un contador interno aumenta con cada `lock()` y disminuye con cada `unlock()`.

**Características principales de ReentrantLock**

1. **Reentrada.** Un mismo hilo puede adquirir el bloqueo varias veces. Para liberarlo por completo debe realizar el mismo número de llamadas a `unlock()`.
2. **Equidad.** El lock puede ser justo o no justo. El modo justo suele dar prioridad al hilo que lleva más tiempo esperando; el modo no justo permite que un hilo nuevo se adelante.
3. **Condition.** `newCondition()` crea condiciones que permiten esperar a que un estado sea válido y recibir una señal cuando cambie.
4. **Espera interrumpible.** `lockInterruptibly()` permite interrumpir un hilo mientras espera el bloqueo.
5. **Intento sin espera.** `tryLock()` responde inmediatamente o espera como máximo el tiempo indicado.

**Uso básico de ReentrantLock**

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

En el ejemplo, `ReentrantLock` protege `increment()` y `getCounter()` frente a accesos simultáneos. `lock()` adquiere el bloqueo y `unlock()` se coloca siempre dentro de `finally`, de modo que el lock se libera aunque la sección crítica lance una excepción. No conviene ejecutar código que pueda fallar entre `lock()` y el inicio del `try`.

**Funciones avanzadas**

1. **Bloqueo justo.** Se crea pasando `true` al constructor:

   ```java
   Lock fairLock = new ReentrantLock(true);
   ```

   Reduce el riesgo de inanición de los hilos que llevan mucho tiempo esperando, aunque puede disminuir el rendimiento global.
2. **Try Lock.** Un intento no bloqueante permite elegir otra estrategia:

   ```java
   if (lock.tryLock()) {
       try {
           // Operaciones protegidas por el lock
       } finally {
           lock.unlock();
       }
   } else {
       // No se pudo adquirir el bloqueo
   }
   ```
3. **Condiciones.** Son útiles para una coordinación más compleja:

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

`await()` libera el lock de forma atómica y suspende el hilo. Después de recibir la señal, el hilo debe recuperar el bloqueo antes de continuar. En código real, la condición se comprueba en un bucle para soportar despertares espurios y cambios realizados por otros hilos.

**Cuándo elegir ReentrantLock en lugar de synchronized**

1. Cuando se necesita un orden justo para reducir la inanición.
2. Cuando los hilos que esperan deben poder ser interrumpidos.
3. Cuando hace falta un intento no bloqueante o con tiempo límite.
4. Cuando se necesitan varias colas `Condition` para un protocolo complejo de espera y notificación.

**Resumen**

`ReentrantLock` es un mecanismo de sincronización potente y flexible. Amplía las posibilidades de `synchronized` con equidad, adquisición interrumpible, `tryLock()` y variables de condición. Esa flexibilidad exige gestionar el ciclo de vida de forma explícita: cada adquisición correcta debe liberarse en `finally`. Utilizado con cuidado, ayuda a construir código concurrente sólido y fácil de mantener.
