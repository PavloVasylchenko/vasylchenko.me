---
title: "De la idea a Google Play: cómo creé un juego con IA durante mis vacaciones"
date: "2026-08-30T00:00:00Z"
draft: false
description: "Cómo el recuerdo de un sencillo juego de estrategia para navegador se convirtió en un experimento con varios modelos de IA, un prototipo funcional y mi propio juego para Android."
slug: "from-flash-game-to-android-game"
categories:
  - "Game Development"
  - "Artificial Intelligence"
tags:
  - "Synaptic Front"
  - "AI-assisted development"
  - "Indie Game"
series: "Synaptic Front with AI"
series_order: 1
---

Cuando estaba en el instituto, por fin tuvimos en casa una conexión a internet bastante rápida para la época. Con ella, los juegos de navegador pasaron a formar parte de mi día a día: no había que instalarlos, cargaban enseguida y eran perfectos para llenar una pausa breve. Mientras el ordenador descomprimía un archivo o instalaba un programa, podía abrir otra pestaña, jugar una partida corta y volver a lo mío unos minutos después. Entre tantos juegos Flash, hubo uno de estrategia que se me quedó especialmente grabado, aunque con los años olvidé su nombre.

El campo de juego estaba formado por sistemas unidos mediante líneas por las que se desplazaban las flotas. La interfaz mostraba de inmediato a quién pertenecía cada nodo, adónde se podían enviar fuerzas y desde dónde se acercaba el enemigo. Cada decisión alteraba el equilibrio: atacar reforzaba un frente, pero debilitaba el sistema de origen, y un nodo conquistado podía convertir una zona segura en una nueva frontera. Las reglas eran sencillas y sus consecuencias se veían en cuestión de segundos. Precisamente por eso, hasta las partidas más cortas mantenían la tensión.

Con el tiempo Flash desapareció, cambiaron los ordenadores y las costumbres, y el nombre del juego no sobrevivió ni en mis marcadores ni en mi memoria. Intenté encontrarlo varias veces a partir de su descripción, pero solo di con proyectos parecidos. La idea, sin embargo, seguía muy viva: una red de nodos conectados, movimientos por rutas definidas y una tensión que nacía de unas pocas reglas comprensibles. Años después, aquel recuerdo se convirtió en el punto de partida de Synaptic Front.

#### De un recuerdo a un juego propio

La idea de crear algo parecido se me ocurrió más de una vez, pero durante mucho tiempo permaneció en esa lista de proyectos para los que nunca hay tiempo. El primer prototipo exigía elegir una tecnología y entender el bucle de juego, la representación del mapa, la animación y los controles. Aunque al final la mecánica no resultara divertida, antes tendría que invertir tiempo en aprender herramientas desconocidas. El experimento salía demasiado caro: varias semanas de trabajo podían terminar en una conclusión a la que quería llegar en una o dos tardes.

Hoy tengo más de quince años de experiencia desarrollando en Java. He trabajado con aplicaciones empresariales y sistemas de servidor de alta carga, así que ni una arquitectura compleja ni un proyecto grande me intimidan. Pero en todos esos años nunca había hecho un juego. Android, Kotlin y Compose tampoco formaban parte de mi trabajo habitual. Sabía programar, pero para esta idea concreta tendría que empezar igualmente en un terreno desconocido.

> **«La IA no inventó este juego: me ayudó a empezar por fin a crearlo».**

En los últimos tiempos, en cambio, he trabajado mucho con herramientas de IA y ya sé bastante bien en qué situaciones ahorran tiempo de verdad. Decidí unir esa experiencia con la vieja idea y averiguar por fin si allí había un juego. Quería llegar cuanto antes a una versión que pudiera ejecutar y valorar por mí mismo, sin dedicar semanas únicamente a familiarizarme con un stack nuevo. En el experimento participaron Claude, GPT, Kimi y el recién aparecido Fable.

Un juego era una prueba mejor que una tarea abstracta. Recordaba la sensación que buscaba, entendía las reglas fundamentales y veía enseguida si cada versión se acercaba a lo que tenía en mente. Lo importante no era cuánto código escribiera un modelo de IA, sino si jugar al resultado sería entretenido.

#### La especificación como referencia común

Por eso el trabajo comenzó describiendo las reglas, no generando código. Preparé unas ideas iniciales y pedí a Fable que las convirtiera en una especificación en Markdown. Así nació `SPEC.md`, el primer documento que recogía el concepto general, la estructura del mapa, las reglas de propiedad de los sistemas, la producción de fuerzas, el movimiento de flotas y las condiciones de conquista. Después, otros modelos analizaron la especificación uno tras otro. Afinaban las formulaciones, detectaban ambigüedades y proponían añadidos; yo conservaba los cambios útiles y descartaba aquello que alejaba el juego de la idea original.

A medida que el concepto crecía, un único `SPEC.md` dejó de ser suficiente. A su lado aparecieron `VISUAL.md`, con las decisiones sobre presentación y animación; `PLOT.md`, con la trama de cada nivel; y `LORE.md`, con el canon general del mundo. Después llegó `CONTENT_PLAN.md`, que transformaba la historia y las ideas de la campaña en un plan técnico de niveles más concreto. Más tarde se sumaron `ECONOMY.md`, dedicado a la evolución de la economía y sus mecánicas, y `OPERATORS.md`, con los héroes operadores que controla el jugador y sus habilidades.

Al principio, esta estructura separaba bien las distintas partes del proyecto, pero los documentos crecieron deprisa y acabaron siendo difíciles de leer. Los dividí en archivos temáticos más pequeños y añadí un `INDEX.md` para navegar por ellos. Así era posible localizar una regla, un elemento de la historia o una habilidad sin cargar todo el material acumulado. Resultó útil tanto para mí como para los modelos: con cada tarea podía entregarles solo la parte pertinente de la documentación.

> **«Cuando el código pasaba de un modelo a otro, la especificación impedía que el proyecto se convirtiera en un caos».**

Este sistema no solo servía como descripción detallada del futuro juego. También separaba las decisiones ya tomadas de las suposiciones de cada modelo. Si la petición se limita a «crea un juego de estrategia con puntos y líneas», el modelo rellenará por su cuenta todos los huecos: decidirá el comportamiento del enemigo, la velocidad de movimiento, el cálculo de resultados y muchos otros detalles. Dos implementaciones no diferirán únicamente en la calidad del código; en la práctica serán juegos distintos.

La documentación reducía las decisiones arbitrarias y permitía devolver cada modelo a la versión acordada del proyecto. Esto fue especialmente valioso cuando el trabajo empezó a circular entre modelos. Ya no tenía que volver a contar todo el contexto: el siguiente recibía la parte necesaria de la especificación, la implementación actual y una tarea concreta. Los documentos tampoco eran un pliego inmutable. Después de probar partidas, ajustaba las reglas y actualizaba su descripción para que el código y la intención siguieran alineados.

Cuando la especificación alcanzó suficiente detalle, pedí a los modelos que crearan las primeras versiones en un único archivo HTML. Elegí ese formato por la velocidad de validación, no pensando en la arquitectura futura. Podía abrir el archivo directamente en el navegador, sin proyecto, compilación, dependencias ni infraestructura adicional. La lógica, la interfaz y la animación cabían juntas, de modo que el resultado de cada iteración aparecía casi al instante.

El experimento nunca pretendió ser un benchmark riguroso. Los modelos no recibieron prompts idénticos, límites fijos ni un sistema formal de puntuación. Las primeras versiones se crearon de forma independiente; después, los límites de las suscripciones y la naturaleza de las tareas hicieron que el trabajo pasara cada vez más de una herramienta a otra. Un modelo preparaba la base, otro corregía un problema y un tercero proponía ampliar una mecánica. Al final comparaba menos los modelos entre sí que las distintas formas de organizar su trabajo sucesivo en un mismo proyecto.

#### El primer prototipo y los ciclos cortos

La mecánica principal seguía siendo compacta. El mapa contenía sistemas conectados por rutas, cada uno de los cuales producía unidades poco a poco. El jugador y el rival controlado por el ordenador enviaban flotas entre nodos vecinos, conquistaban posiciones y trataban de conservarlas. Para atacar con éxito no bastaba con elegir un objetivo débil: había que pensar cuánto debilitaría la defensa el envío de una flota y si el rival podría aprovechar la ruta que quedaba abierta.

La primera versión convincente llegó cuando la flota elegida recorrió realmente la línea hasta el sistema indicado. La animación era mejor de lo que esperaba de un prototipo HTML temprano, pero había algo más importante: por fin podía evaluar la mecánica en acción. Hasta entonces solo existían un recuerdo, una especificación y unas reglas. Ahora podía hacer un movimiento, observar sus consecuencias y comprobar si el mapa generaba aquella tensión que había dado origen al experimento.

El prototipo confirmó que merecía la pena continuar y comenzaron ciclos de desarrollo cortos. Aclaraba una regla en la especificación, daba al modelo una tarea acotada, ejecutaba la nueva versión y jugaba varias partidas. Si el comportamiento no era el esperado, la diferencia se convertía en la siguiente tarea. Este proceso aportaba más que la mera velocidad de generación, porque cada cambio llegaba rápidamente a un resultado comprobable.

Durante esas iteraciones podía modificar el ritmo de producción, la velocidad de las flotas y el equilibrio entre ataque y defensa, y observar al instante cómo afectaban los nuevos valores a la partida. Poco a poco entendí qué dificultad obligaba a buscar una solución y cuál se percibía como injusta. El viejo recuerdo marcaba el rumbo, pero las reglas concretas de Synaptic Front nacieron durante las pruebas.

La siguiente herramienta auxiliar fue un editor de niveles provisional. La primera pantalla podía describirse a mano en los datos, pero una prueba completa requería distintas configuraciones de nodos y rutas. El editor permitió crearlas con rapidez y comprobar si el interés sobrevivía más allá de un único diseño acertado. No estaba destinado a usuarios ni necesitaba un acabado de producto, pero aceleró mucho el ciclo principal. Para esta clase de herramientas internas, la generación con IA resultó especialmente útil: el pequeño coste de implementarlas se recuperaba en las iteraciones siguientes.

#### Del HTML a Android

Tras varias versiones funcionales quedó claro que la idea había superado la prueba inicial. Al mismo tiempo apareció un reto más difícil: llevar el proceso más allá de un archivo HTML autónomo. Android me resultaba más cercano que iOS, así que decidí crear la siguiente versión con Kotlin y Compose. Ese paso trazó la frontera entre un experimento rápido y el desarrollo de una aplicación real.

> **«HTML demostró que la idea funcionaba; Android tenía que convertirla en un producto».**

Para entonces ya había probado varios modelos. Fable fue el primero: acababa de salir y quería ver cómo respondía ante una tarea real. Según mi experiencia, GPT-5.5, disponible entonces en Codex, estaba por detrás, así que apenas usé Codex. Claude siguió siendo mi herramienta principal durante las primeras fases del desarrollo para Android.

En el prototipo web, la lógica, la representación y el estado podían convivir, y una versión fallida se sustituía entera con facilidad. El proyecto Android introdujo el ciclo de vida de la aplicación, la persistencia de datos, la gestión del estado, las pruebas y la necesidad de comprobar el comportamiento en un dispositivo real. Una pantalla que funcionaba seguía siendo un resultado importante, pero ya no demostraba que el producto estuviera listo. Cada cambio debía valorarse dentro de la estructura existente y por su efecto sobre el resto de la aplicación.

La situación cambió con la llegada de Sol. El nuevo modelo trabajaba con mucha más seguridad que 5.5 y a un nivel comparable al de Fable, así que cancelé mi suscripción a Claude y decidí continuar en Codex. Para entonces había acumulado tres restablecimientos de límite, y la suscripción Plus bastaba para avanzar sin prisas. Construí la mayor parte de la aplicación así: planteaba la siguiente tarea, revisaba el resultado y continuaba cuando el límite volvía a estar disponible.

Al igual que Claude, Codex podía iniciar el emulador de Android, abrir la aplicación, tomar capturas y comprobar el resultado de un cambio. Si una pantalla se mostraba mal o fallaba un escenario, el modelo podía ver el problema, volver al código, corregirlo y repetir la prueba. Para Android, esto era mucho más útil que generar archivos sin más: gran parte del ciclo entre la modificación y la comprobación visual transcurría sin cambiar manualmente de herramienta.

Kimi K3 apareció hacia la mitad del desarrollo y empecé a encargarle algunas tareas. La parte posterior de la aplicación se construyó así entre varios: unas piezas las escribió Codex con Sol y otras Kimi K3. Los servicios agotaban sus límites en momentos distintos y, en vez de esperar, yo cambiaba de modelo. Era también otra prueba del proceso: ¿hasta qué punto podía una herramienta nueva continuar el proyecto de la anterior?

Cada modelo leía y aclaraba primero la documentación y solo después modificaba el código. Si aparecía una restricción nueva o se tomaba otra decisión, volvía a los documentos y pasaba a formar parte del contexto común. El traspaso funcionaba en ambos sentidos: el siguiente modelo recibía el conocimiento acumulado y dejaba una descripción más precisa para el que viniera después. Poco a poco dejó de tener sentido elegir un único modelo «mejor». Importaban mucho más la calidad de la documentación, el tamaño de la tarea y la capacidad de verificar el cambio de forma independiente.

#### Cómo cambia el papel de la IA al crecer el proyecto

En el prototipo inicial, equivocarse costaba poco. Si una versión no funcionaba, podía descartarla después de una sesión. Por eso era posible entregar grandes partes de la implementación a los modelos y explorar variantes rápidamente. En el proyecto Android, ese mismo nivel de confianza empezó a frenar el trabajo. El código crecía, aparecían relaciones entre componentes y corregir un problema podía romper una premisa de otra parte de la aplicación.

> **«Cuanto más crecía el proyecto, menos trabajo podía delegar en la IA sin cuestionarlo».**

Con el tiempo, el papel de la IA pasó de generar implementaciones enteras a prestar apoyo de ingeniería. Resultaba más útil discutir varias soluciones, comprobar hipótesis, localizar el origen de una discrepancia y preparar cambios pequeños y controlados. Cada paso importante seguía necesitando pruebas y revisión manual, y la decisión final correspondía a la persona que entendía el producto y respondía por el resultado.

Mantener sincronizados el código y la documentación también se volvió esencial. Si corregía un comportamiento a mano, no bastaba con guardar el cambio en el repositorio. La nueva regla o decisión arquitectónica debía reflejarse en los documentos que recibían los modelos. De lo contrario, en pocas iteraciones la persona y la IA trabajaban con versiones distintas: el modelo recuperaba una limitación antigua o intentaba restaurar una decisión descartada, y el tiempo se perdía resolviendo contradicciones evitables.

Esta experiencia no demostró que la IA vuelva sencillo el desarrollo de juegos ni que permita ignorar un stack nuevo. Su valor fue otro: me permitió llegar a la primera versión comprobable antes de que el coste de empezar me hiciera abandonar la idea.

Después del prototipo HTML, el problema cambió. Las dudas sobre la mecánica dieron paso a las preguntas habituales de un producto: cómo mantener el código, dónde trazar los límites del estado, cómo verificar los cambios y cómo conservar el conocimiento de las decisiones tomadas. Así, un experimento con varios modelos y archivos HTML autónomos se convirtió poco a poco en un proyecto Android, y el recuerdo de un juego Flash sin nombre, en Synaptic Front.

> **Prueba Synaptic Front**
>
> El juego ya está disponible en [Google Play](https://play.google.com/store/apps/details?id=me.vasylchenko.synapticfront). Allí puedes ver en qué se convirtió la idea de esta historia y jugarlo tú mismo.
>
> *Continuará.*
