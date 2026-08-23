---
title: "Uso de Semaphore en Java"
date: 2024-06-20T08:16:17Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Qué es Semaphore en Java**

`Semaphore` es una ayuda de sincronización que limita cuántos hilos pueden acceder simultáneamente a un recurso compartido. Forma parte de `java.util.concurrent` y resulta especialmente útil para recursos con capacidad fija, como pools de conexiones, sockets de red, impresoras o descriptores de archivo.

**Características principales de Semaphore**

1. **Permisos.** El semáforo mantiene un conjunto de permits. Antes de trabajar, un hilo adquiere uno y lo devuelve al terminar.
2. **Adquisición bloqueante y no bloqueante.** `acquire()` espera a que exista un permiso; `tryAcquire()` permite seguir de inmediato por una ruta alternativa.
3. **Equidad.** Un semáforo puede ser justo o no justo. El modo justo suele conceder permisos en el orden de solicitud y reduce el riesgo de inanición.
4. **Varios permisos.** Un hilo puede adquirir y liberar más de un permit si una operación consume varias unidades del recurso.

**Ejemplo básico de Semaphore**

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

            // Simulamos trabajo con el recurso compartido
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

El semáforo se crea con tres permisos. Se inician diez hilos, pero solo tres pueden superar `acquire()` y ejecutar la sección crítica a la vez. Los demás esperan. Cuando uno llama a `release()`, otro participante puede adquirir el permiso liberado.

La variable `acquired` es importante: si el hilo fue interrumpido antes de adquirir el permiso, no debe ejecutar `release()`, porque aumentaría incorrectamente el número total.

**Uso avanzado de Semaphore**

1. **Intento sin bloqueo.** El hilo comprueba el recurso y elige inmediatamente otra estrategia si no está disponible:

   ```java
   if (semaphore.tryAcquire()) {
       try {
           // Sección crítica
       } finally {
           semaphore.release();
       }
   } else {
       System.out.println(Thread.currentThread().getName() + " could not acquire a permit.");
   }
   ```
2. **Adquisición con timeout.** Limitar la espera evita bloqueos indefinidos durante una sobrecarga:

   ```java
   if (semaphore.tryAcquire(1, TimeUnit.SECONDS)) {
       try {
           // Sección crítica
       } finally {
           semaphore.release();
       }
   } else {
       System.out.println(Thread.currentThread().getName() + " timed out waiting for a permit.");
   }
   ```
3. **Semáforo justo.** El segundo argumento activa el orden de espera:

   ```java
   Semaphore fairSemaphore = new Semaphore(MAX_PERMITS, true);
   ```

La equidad es útil cuando ningún hilo debe esperar demasiado, aunque mantener la cola puede reducir el rendimiento total.

**Casos de uso**

1. **Limitar el paralelismo.** Controlar el número de solicitudes simultáneas a un servicio externo o a una operación costosa.
2. **Pools de conexiones.** Cada permiso representa una conexión disponible con la base de datos.
3. **Gestión de recursos.** Restringir el acceso a un conjunto pequeño de impresoras, archivos o dispositivos.
4. **Protección frente a sobrecarga.** Las tareas adicionales esperan o se rechazan en lugar de consumir memoria y CPU al mismo tiempo.

**Resumen**

`Semaphore` ofrece una forma sólida de limitar el acceso concurrente a recursos compartidos. A diferencia de un mutex, puede permitir que un número definido de hilos avance simultáneamente. Es imprescindible liberar únicamente los permisos adquiridos y hacerlo siempre en `finally`. Utilizado correctamente, ayuda a aprovechar recursos escasos y a reducir la contención en aplicaciones multihilo.
