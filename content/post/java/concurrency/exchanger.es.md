---
title: "Cómo funciona Exchanger en Java"
date: 2024-06-21T07:16:11Z
tags:
  - "Java"
  - "Concurrency"
categories:
  - "Java/Concurrency"
draft: false
---

**Qué es Exchanger en Java**

La clase `Exchanger` crea un punto de sincronización donde dos hilos forman una pareja e intercambian objetos. Cada hilo entrega su valor a `exchange()`, espera a su compañero y, cuando ambos coinciden, recibe el objeto proporcionado por el otro.

El mecanismo resulta útil cuando dos hilos necesitan pasarse datos directamente. Por ejemplo, un productor puede llenar un búfer mientras un consumidor procesa el anterior y, al terminar, ambos intercambian los búferes sin utilizar una cola común. `Exchanger` se ocupa al mismo tiempo de transferir el valor y sincronizar el instante del intercambio.

**Características principales de Exchanger**

1. **Intercambio de objetos.** Dos hilos entregan y reciben valores del mismo tipo genérico.
2. **Operación bloqueante.** `exchange()` espera hasta que otro hilo alcanza el mismo punto.
3. **Tiempo límite.** Una sobrecarga del método permite limitar la espera y evitar un bloqueo permanente.
4. **Trabajo por parejas.** Si llegan más de dos hilos, `Exchanger` los empareja; no se puede seleccionar de antemano un compañero concreto.

**Uso básico de Exchanger**

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

En el ejemplo:

1. **Inicialización.** Se crea `Exchanger<String>`, por lo que los participantes intercambian cadenas.
2. **Producer y Consumer.** Ambos implementan `Runnable`, preparan sus datos y llaman al mismo objeto mediante `exchanger.exchange()`.
3. **Ejecución.** El primer hilo que llega queda bloqueado. Cuando aparece el segundo, los valores se intercambian de forma atómica y los dos continúan.

Si uno de los participantes es interrumpido durante la espera, `exchange()` lanza `InterruptedException`. El código restaura el estado de interrupción para que la capa superior pueda gestionar la cancelación correctamente.

**Exchanger con timeout**

Si existe la posibilidad de que el compañero no llegue, no conviene esperar indefinidamente. La versión con tiempo límite permite finalizar la operación de forma predecible.

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

Los dos hilos esperan como máximo cinco segundos. Si el compañero no aparece, se produce `TimeoutException` y la aplicación puede reintentar, liberar recursos o cancelar la tarea. Conviene tratar timeout e interrupción por separado: el timeout puede ser un resultado esperado del protocolo, mientras que una interrupción suele indicar una solicitud de cancelación.

**Cuándo utilizar Exchanger**

- para intercambiar un búfer lleno por otro vacío;
- para comparar o combinar resultados intermedios de dos trabajadores;
- en una tubería donde dos etapas deben encontrarse al final de cada ronda;
- en pruebas de concurrencia que requieren un punto exacto de sincronización entre dos hilos.

Con varios productores y consumidores suele ser más apropiado `BlockingQueue`. `Exchanger` funciona mejor cuando el protocolo está formado naturalmente por parejas.

**Resumen**

`Exchanger` combina la transferencia de datos y la sincronización de dos hilos en una sola operación. Hace explícita la comunicación por parejas y evita crear un contenedor compartido independiente. El timeout protege frente a esperas infinitas, y una gestión correcta de las interrupciones permite cancelar el trabajo. En los escenarios adecuados simplifica la coordinación y mejora la claridad del algoritmo concurrente.
