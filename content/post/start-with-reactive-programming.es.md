---
title: "Primeros pasos con la programación reactiva"
date: 2022-08-04T22:00:00Z
categories:
  - "Manuals"
draft: false
---

Hoy quiero hablar de los flujos reactivos.
Este enfoque ha ganado mucha popularidad en los últimos años. Spring permite utilizarlo como alternativa al modelo basado en servlets.
Cada vez más bibliotecas cliente ofrecen una API reactiva, y las bases de datos admiten conexiones mediante R2DBC y controladores reactivos.
Yo también empleo Project Reactor con mayor frecuencia en mis módulos.

Los flujos reactivos suelen compararse con Java Streams porque ambos están relacionados con la programación funcional y comparten una sintaxis parecida.
Sin embargo, un `Stream` normal solo puede consumirse una vez, por lo que los desarrolladores rara vez devuelven valores como `Stream<String>` desde sus métodos.
Lo habitual es construir toda la cadena en una sola función. Esto resulta cómodo para transformaciones pequeñas, pero las llamadas siguen siendo bloqueantes.

Hace unos meses decidí realizar un curso de Project Reactor en Udemy para ordenar mis conocimientos y descubrir ideas nuevas.
El curso fue interesante y útil, sobre todo para quienes empiezan a trabajar con el paradigma reactivo.

Después preparé una implementación del «Juego de la vida» basada en flujos reactivos.
Es un caso muy apropiado: la fuente puede generar una secuencia ilimitada y el consumidor decide posteriormente cuántos elementos necesita.
El código está disponible en el repositorio [GameOfLife](https://github.com/PavloVasylchenko/GameOfLife).

El proyecto se divide en varias partes:

- `CellState`, una enumeración que indica si una celda está viva o muerta;
- `Game`, que contiene la lógica principal y el generador de ciclos;
- `UI`, responsable de imprimir el tablero en la consola.

El campo se representa mediante una matriz bidimensional `CellState[][]`.
Un único método público recibe el estado inicial y devuelve un `Flux` con los estados siguientes calculados a partir de él:

```java
Flux<CellState[][]> game(CellState[][] initialField)
```

El método no impone un número de iteraciones: esa decisión corresponde al cliente.
El flujo se construye en dos pasos:

1. `Mono.just(initialField)` coloca el tablero inicial al principio de la secuencia.
2. `.concatWith(generate)` añade el generador de las generaciones siguientes.

```java
Flux<CellState[][]> generate = Flux.generate(() -> initialField, (state, sink) -> {
    CellState[][] iterate = iterate(state);
    sink.next(iterate);
    return iterate;
});
```

El generador calcula un nuevo tablero a partir del anterior, lo envía al `Flux` actual y lo conserva como estado para el paso siguiente.

El test muestra cómo consumir esta lógica:

```java
@Test
public void printerTest() {
    new Game().game(getGliderField())
            .take(5)
            .doOnNext(UI::printState)
            .subscribe();
}
```

La partida comienza con un patrón «glider», limita el flujo a cinco elementos e imprime cada tablero.
El generador solo se ejecuta cuatro veces porque el estado inicial ya ocupa la primera posición.
Si necesitamos más generaciones o algún procesamiento adicional, basta con añadir nuevos operadores al `Flux` existente.
