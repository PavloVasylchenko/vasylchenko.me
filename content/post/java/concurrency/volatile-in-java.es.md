---
title: "Cómo funciona la palabra clave volatile en Java"
date: 2024-06-17T10:00:00Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

La palabra clave `volatile` de Java indica que el valor de una variable puede ser modificado por distintos hilos. Al declarar una variable como `volatile`, los hilos deben observar su valor actualizado en lugar de seguir utilizando una copia antigua almacenada localmente. Esta garantía es esencial en código concurrente, donde un hilo podría no detectar el cambio realizado por otro.

**¿Por qué utilizar volatile?**

Para mejorar el rendimiento, cada hilo y cada procesador pueden mantener valores en registros o cachés. Como consecuencia, un hilo podría continuar trabajando con un dato obsoleto incluso después de que otro haya escrito un valor nuevo. `volatile` establece reglas de visibilidad: una escritura publica el cambio y una lectura posterior obtiene el valor actualizado.

**Características principales de volatile**

1. **Visibilidad.** Los cambios de una variable `volatile` pasan a ser visibles para los demás hilos. La escritura no puede permanecer únicamente en la vista local del hilo que la realizó.
2. **Atomicidad.** `volatile` no convierte en atómicas las operaciones compuestas. Por ejemplo, `counter++` consta de una lectura, un incremento y una escritura; dos hilos pueden interferir y perder una actualización.
3. **Orden.** El Java Memory Model limita la reordenación de lecturas y escrituras alrededor de `volatile`. Una escritura happens-before respecto a una lectura posterior de la misma variable desde otro hilo.

**Cuándo utilizar volatile**

1. Cuando una variable sencilla se comparte entre varios hilos.
2. Cuando un lector debe observar el último valor publicado por otro hilo.
3. Cuando el estado cambia mediante una sola asignación y no forma parte de operaciones read-modify-write, check-then-act o invariantes complejos.

Los casos habituales incluyen una bandera de parada, un indicador de disponibilidad o una referencia a una configuración inmutable.

**Ejemplo de volatile**

```java
public class VolatileExample {
    private volatile boolean flag = false;

    public void writer() {
        flag = true;  // escritura en una variable volatile
    }

    public void reader() {
        if (flag) {  // lectura de la variable volatile
            System.out.println("Flag is true!");
        }
    }

    public static void main(String[] args) {
        VolatileExample example = new VolatileExample();

        Thread writerThread = new Thread(() -> {
            try {
                Thread.sleep(1000); // simulamos algo de trabajo
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            example.writer();
            System.out.println("Flag set to true by writer thread");
        });

        Thread readerThread = new Thread(() -> {
            while (!example.flag) {
                // esperamos hasta que la bandera sea true
            }
            example.reader();
        });

        writerThread.start();
        readerThread.start();
    }
}
```

En este ejemplo, `flag` está declarada como `volatile`. El hilo escritor espera un segundo y asigna `true`. El hilo lector comprueba la bandera en un bucle y, gracias a la garantía de visibilidad, detecta el nuevo valor. Entonces sale del bucle y ejecuta `reader()`.

Sin `volatile`, el Java Memory Model no obliga al lector a volver a obtener el valor compartido. En teoría, podría continuar observando `false` indefinidamente. El bucle de espera activa tampoco sería una buena solución en una aplicación real; aquí solo sirve para mostrar claramente el problema de visibilidad.

**Limitaciones de volatile**

1. **No ofrece atomicidad para acciones compuestas.** Un incremento, una comparación seguida de una escritura o la actualización coordinada de varios campos requieren otro mecanismo.
2. **No protege estados complejos.** Si la corrección depende de que varios valores sean coherentes, conviene utilizar `synchronized`, `ReentrantLock` o una estructura de datos segura para hilos.
3. **No sustituye la coordinación.** Para esperar eventos son preferibles `CountDownLatch`, las condiciones de un lock, las colas u otras abstracciones de alto nivel.

**Resumen**

`volatile` es una herramienta sencilla para publicar estados simples entre hilos. Garantiza visibilidad y cierto orden de memoria, pero no hace atómicas las acciones compuestas. Suele bastar para banderas y asignaciones independientes; los contadores y las transiciones complejas necesitan clases atómicas, bloqueos o secciones sincronizadas. Elegir el mecanismo adecuado ayuda a mantener el código concurrente correcto y comprensible.
