---
title: "Clases atómicas en Java"
date: 2024-06-18T11:12:01Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

Para construir aplicaciones multihilo robustas es imprescindible gestionar los datos compartidos de forma correcta y eficiente. En Java se utilizan con frecuencia variables `volatile` y clases atómicas del paquete `java.util.concurrent.atomic`. Ambos mecanismos permiten que varios hilos trabajen con un mismo estado, pero persiguen objetivos diferentes.

**¿Por qué utilizar clases atómicas?**

Las clases atómicas permiten ejecutar operaciones indivisibles sobre una sola variable sin recurrir a sincronización explícita. Un incremento, un decremento, una suma o una operación compare-and-set (CAS) se completan como un único paso lógico: ningún otro hilo puede observar un estado intermedio ni intervenir entre la lectura y la escritura.

El paquete `java.util.concurrent.atomic` incluye, entre otras, las siguientes clases:

- `AtomicInteger`;
- `AtomicLong`;
- `AtomicBoolean`;
- `AtomicReference`.

Todas ellas ofrecen métodos seguros para leer, modificar y actualizar valores de manera condicional. En escenarios sencillos, permiten evitar `synchronized` y los bloqueos explícitos.

**Características principales de las clases atómicas**

- **Atomicidad.** Una operación es indivisible: los demás hilos solo observan el estado anterior o el posterior.
- **Ejecución no bloqueante.** La mayoría de los métodos se apoya en instrucciones de bajo nivel del procesador y no suspende el hilo mientras espera un monitor.
- **Enfoque lock-free.** Normalmente no se usa exclusión mutua, lo que reduce el riesgo de esperas prolongadas y deadlocks.
- **Visibilidad.** Las lecturas y escrituras respetan las garantías necesarias del Java Memory Model, por lo que otros hilos pueden observar el resultado.

Esto no significa que las clases atómicas siempre sean más rápidas que un lock. Cuando existe mucha contención, los reintentos de CAS también pueden resultar costosos. La elección depende del algoritmo y de la carga real.

**Ejemplo con AtomicInteger**

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicExample {
    private AtomicInteger counter = new AtomicInteger(0);

    public void increment() {
        counter.incrementAndGet();  // incremento atómico
    }

    public int getValue() {
        return counter.get();
    }

    public static void main(String[] args) {
        AtomicExample example = new AtomicExample();

        // Creamos 1000 hilos que incrementan el contador
        Thread[] threads = new Thread[1000];
        for (int i = 0; i < 1000; i++) {
            threads[i] = new Thread(example::increment);
            threads[i].start();
        }

        // Esperamos a que todos terminen
        for (Thread thread : threads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }

        System.out.println("Final counter value: " + example.getValue());
    }
}
```

`incrementAndGet()` realiza la lectura, el incremento y la escritura como una única operación atómica. Por eso, cuando finalizan los 1000 hilos, el contador contiene el valor esperado. Un `int` normal, incluso si fuese `volatile`, podría perder actualizaciones al ejecutar `++` de forma concurrente.

**Diferencias entre volatile y las clases atómicas**

1. **Visibilidad frente a atomicidad.**
   - `volatile` garantiza que los hilos vean las escrituras recientes, pero no protege acciones compuestas.
   - Las clases atómicas proporcionan visibilidad y atomicidad para las operaciones que exponen.
2. **Actualizaciones complejas.**
   - `volatile` es adecuado para lecturas y asignaciones independientes.
   - Las clases atómicas ofrecen `incrementAndGet()`, `decrementAndGet()`, `addAndGet()` y `compareAndSet()` para operaciones read-modify-write seguras.
3. **Implementación.**
   - `volatile` utiliza las reglas del Java Memory Model para publicar y ordenar accesos.
   - Las clases atómicas suelen emplear CAS por hardware y repiten la operación cuando es necesario.
4. **Casos de uso.**
   - `volatile` se utiliza para banderas, indicadores de disponibilidad y estados simples.
   - Las clases atómicas son apropiadas para contadores, referencias y valores actualizados con frecuencia por varios hilos.

**Resumen**

Comprender la diferencia entre `volatile` y las clases atómicas permite escoger el mecanismo mínimo necesario. `volatile` resuelve la visibilidad, mientras que las clases atómicas añaden operaciones compuestas indivisibles. Cuando el invariante afecta a una sola variable, suelen ofrecer una solución sencilla y no bloqueante. Si es necesario modificar varios valores de forma coordinada, ejecutar una sección crítica extensa o esperar una condición, conviene recurrir a locks u otras abstracciones de concurrencia de nivel superior.
