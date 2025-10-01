---
title: "Getting started with Reactive Programming"
date: 2022-08-04T22:00:00Z
categories: 
    - "Manuals"
draft: false
---

Hello, today I want to talk about reactive streams. 
This framework becomes popular last days. Spring allow to use it instead of servlet based approach. 
Also clients more and more often support reactive api nowadays. Databases allow connect via r2dbc and reactive drivers. 
I use project reactor in my modules more and more often. 

Often people use Java streams since they are similar and also related to functional programming but their usage is limited. They might be consumed only one time, so devs rarely return something like `Stream<String>` from functions. Usually they put all pipeline in same function what is convenient for small data processing but eventually it is blocking calls.

Few months ago I decided to watch course on Udemy related to project reactor to collect all thoughts together and find some new ideas.
If general it was very interesting and useful to watch these courses. It will be the most useful for developers new to reactive approach.

As result, I decided to prepare project “game of life” based on reactive streams. It is good case for reactive project because you can create unlimited generator and limit it later by consuming it.
Here is project link https://github.com/PavloVasylchenko/GameOfLife

So we have the following structure:
CellState which is enum and it shows if cell dead or alive
Game class which contains main logic an game cycle generator
UI class which is responsible for printing field to console

Filed represented as array of arrays of CellState (CellState[][])
We have just one method that will receive initial state and then provide flux with next states calculated based on initial state.

```
Flux<CellState[][]> game(CellState[][] initialField)
```

This is interesting part because it does not receive some limits but client can decide about amount of iterations.
There are 2 steps:
1. `Mono.just(initialField)` where we take initial field and return it at the beginning of flow.
2. `.concatWith(generate)` where we contact it with next fields generator 

```
Flux<CellState[][]> generate = Flux.generate(() -> initialField, (state, sink) -> {
             CellState[][] iterate = iterate(state);
             sink.next(iterate);
             return iterate;
         });
```
        
In generator we generate new field based on previous one and then send it to the current Flux.

GameTest show usage of this logic.
For example we have 

```
@Test
public void printerTest() {
    new Game().game(getGliderField())
            .take(5)
            .doOnNext(UI::printState)
            .subscribe();
}
```

We are starting game with initial glider pattern and then limit amount of iterations to 5. Then we print resulting field.
So logic that generates field will be executed only 4 times + initial field will be added to the head of the list. If we need more items or if we need some processing we can add these steps to existing Flux.
