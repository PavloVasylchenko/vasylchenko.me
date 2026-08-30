---
title: "From Idea to Google Play: How I Built a Game with AI While on Vacation"
date: "2026-08-30T00:00:00Z"
draft: false
description: "How a memory of a simple browser strategy game became a multi-model AI experiment, a working prototype, and an Android game of my own."
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

When I was in high school, we finally got what counted as fast internet at home. Browser games soon became part of everyday life: there was nothing to install, they loaded quickly, and they were perfect for filling short pauses. While an archive was unpacking or a program was installing, I could open a new tab, play a quick match, and get back to whatever I was doing a few minutes later. Of all those Flash games, one strategy game stayed with me in particular, even though I eventually forgot its name.

The playing field was made up of systems connected by lines, with fleets travelling between them. At a glance, the interface showed who controlled each node, where forces could be sent, and which direction the enemy was approaching from. Every decision shifted the balance: an attack strengthened one front but weakened the system it launched from, while a captured node could turn a safe area into a new border. The rules were simple and the consequences appeared within seconds. That was exactly what made even short matches tense.

Flash eventually disappeared, computers and habits changed, and the game's name survived neither in my bookmarks nor in my memory. I tried several times to find it from a description, but only ever found similar projects. The idea itself, however, remained vivid: a network of connected nodes, movement along fixed routes, and tension emerging from a handful of clear rules. Years later, that memory became the starting point for Synaptic Front.

#### From a memory to a game of my own

I had thought about making a similar game more than once, but for a long time it remained on the list of ideas I never got around to. A first prototype would mean choosing a technology and figuring out the game loop, map rendering, animation, and controls. Even if the mechanic turned out not to be fun, I would first have to spend time learning unfamiliar tools. That made the experiment too expensive: several weeks of work could end with a conclusion I wanted to reach in one or two evenings.

I now have more than 15 years of Java development experience. Over that time I have worked on enterprise applications and high-load server systems, so complex architecture and large projects do not intimidate me. But I had never made a game. Android, Kotlin, and Compose were outside my usual work too. I knew how to program, yet this particular idea would still require me to begin in unfamiliar territory.

> **“AI did not invent this game—it helped me finally start making it.”**

Lately, though, I had been working extensively with AI tools and had developed a good sense of where they genuinely save time. I decided to combine that experience with the old idea and finally find out whether there was a game in it. I wanted to reach a version I could launch and judge for myself as quickly as possible, without spending weeks merely getting acquainted with a new stack. Claude, GPT, Kimi, and the newly released Fable all took part in the experiment.

A game was a better test for this than an abstract sample task. I remembered the feeling I was after, understood the core rules, and could immediately tell whether each new version was getting closer to what I had in mind. What mattered was not how much code an AI model could write, but whether the resulting game was actually enjoyable to play.

#### A specification as a shared point of reference

That is why the work began with the rules, not code generation. I wrote a handful of initial ideas and asked Fable to turn them into a Markdown specification. The result was `SPEC.md`, the first document to describe the overall concept, map structure, system ownership, force production, fleet movement, and capture conditions. Other models then reviewed the specification in turn. They sharpened the wording, found ambiguous areas, and suggested additions; I kept the useful changes and removed anything that pulled the game away from the original idea.

As the concept developed, a single `SPEC.md` was no longer enough. It was joined by `VISUAL.md`, which collected decisions about presentation and animation; `PLOT.md`, with the stories of individual levels; and `LORE.md`, containing the broader canon of the game world. `CONTENT_PLAN.md` came next, turning the campaign's story and ideas into a more concrete technical plan for its levels. Later, the group grew to include `ECONOMY.md`, covering the future development of the economy and related mechanics, and `OPERATORS.md`, describing the operator heroes controlled by the player and their abilities.

At first this structure separated the different parts of the project well, but the documents grew quickly and became cumbersome in their own right. I split them into smaller thematic files and added an `INDEX.md` for navigation. A particular rule, plot point, or ability description could then be found on its own, without loading the entire body of material every time. That helped both me and the models: I could provide only the relevant portion of the documentation with each task.

> **“When the code passed from one model to another, the specification kept the project from descending into chaos.”**

This document system mattered for more than providing a detailed description of the future game. It separated decisions we had already made from the assumptions of any particular model. Give a model nothing more than “make a strategy game with dots and lines,” and it will fill every gap itself: enemy behaviour, movement speed, how results are calculated, and countless other details. Two implementations would then differ not only in code quality; they would effectively be different games.

The documentation reduced arbitrary decisions and gave me a way to bring each model back to the agreed version of the project. This became especially valuable once work started moving between models. I no longer had to retell the context each time: the next model received the relevant part of the specification, the current implementation, and one concrete task. The documents were not an immutable requirements contract, however. After playtesting, I adjusted the rules and then updated their descriptions so that the code and the intent continued to match.

Once the specification was detailed enough, I asked the models to build the first versions of the game as single HTML files. I chose that format for validation speed, not future architecture. A file could be opened directly in a browser, with no project, build, dependencies, or separate infrastructure. It could contain the game logic, interface, and animation together, which made the result of each iteration visible almost immediately.

The experiment was never meant to be a rigorous benchmark. The models did not receive identical prompts, fixed limits, or a formal scoring system. The first versions were built independently; later, subscription limits and the nature of individual tasks meant that work moved more and more often from one tool to another. One model might lay the groundwork, the next fix a problem I had found, and a third suggest how to extend a mechanic. In the end, I was comparing not so much the models themselves as different ways to organise their sequential work on a single project.

#### The first prototype and short development loops

The prototype's core mechanic remained compact. The map contained systems connected by routes, each gradually producing units. The player and the computer opponent sent fleets between neighbouring nodes, captured new positions, and tried to hold those they already controlled. A successful attack required more than choosing a weak target: I had to consider how much sending a fleet would weaken my defence and whether the opponent could exploit the newly opened route.

The first convincing version arrived when the selected fleet actually travelled along a line to the target system. The animation looked better than I had expected from an early HTML prototype, but something else mattered more: I could finally judge the mechanic in action. Until then there had only been a memory, a specification, and a collection of rules. Now I could make a move, see the consequences, and find out whether the map produced the tension that had prompted the whole experiment.

The prototype confirmed that the work was worth continuing, and the process shifted into short development loops. I refined a rule in the specification, gave a model a bounded task, launched the new version, and played several matches. If its behaviour differed from what I expected, that discrepancy became the next task. This process proved more useful than raw code-generation speed because every change quickly reached a result I could test.

During these iterations I could change the pace of force production, fleet speed, and the balance between attack and defence, then immediately see how the new values affected a match. Gradually, I learned which kinds of difficulty made me search for a solution and which simply felt unfair. The old memory pointed the way, but the specific rules of Synaptic Front took shape through playtesting.

A rough level editor became the next supporting tool. I could describe the first map by hand in the data, but a proper test required different arrangements of nodes and routes. The editor made it faster to create those variants and determine whether the game remained interesting beyond one successful layout. It was not intended for players and needed no product polish, yet it noticeably accelerated the main loop. AI generation was particularly useful for tools like this: a modest implementation effort paid for itself in the very next iterations.

#### Moving from HTML to Android

After several working versions, it was clear that the idea had passed its initial test. At the same time, a harder challenge emerged: taking the process beyond a self-contained HTML file. Android was more familiar to me than iOS, so I decided to build the next version in Kotlin and Compose. That transition marked the line between a quick experiment and the development of a real application.

> **“HTML proved that the idea worked; Android had to turn it into a product.”**

By then I had already tried several models on the project. Fable came first: it had just been released, and I wanted to see how it would handle a real task. In my experience, GPT-5.5, the model available in Codex at the time, lagged behind it, so I barely used Codex itself. Claude remained my main tool during the early stages of Android development.

In the browser prototype, game logic, rendering, and state could all sit side by side, and a failed version was easy to replace wholesale. The Android project introduced an application lifecycle, data persistence, state management, tests, and the need to verify behaviour on a real device. A working screen was still an important result, but it was no longer evidence of a finished product. Every change had to be evaluated in the context of the existing structure and its effect on the rest of the application.

The situation changed with the release of Sol. The new model worked much more confidently than 5.5 and was comparable to Fable, so I cancelled my Claude subscription and decided to continue development in Codex. By then I had accumulated three limit resets, and a Plus subscription was enough to keep the project moving without rushing. I built most of the Android application in that rhythm: set the next task, check the result, and continue when the limit became available again.

Like Claude, Codex could launch the Android emulator, open the application, take screenshots, and verify the result of a change on its own. If a screen rendered incorrectly or a scenario failed, the model could see the problem in the emulator, return to the code, make a correction, and test it again. For Android development, this was far more useful than simply generating files: a large part of the cycle from modification to visual verification happened without manual switching between tools.

Kimi K3 appeared around the middle of development, and I began giving it some of the tasks. The later part of the Android application was therefore built collaboratively: some pieces by Codex with Sol, others by Kimi K3. The services reached their limits at different times, so instead of waiting I moved to another model. This also became another test of the process: how well could a new tool continue a project after the previous one?

Each model first read and clarified the documentation, then changed the code. If the work revealed a new constraint or led to a different decision, it went back into the documents and became part of the shared context. The hand-off therefore worked both ways: a model received the accumulated knowledge and left behind a more accurate description for whichever model came next. Gradually, choosing a single “best” model stopped making sense. Documentation quality, task size, and the ability to verify a change independently mattered far more.

#### How AI's role changes as a project grows

The cost of a mistake was low during the early prototype. If a version worked poorly, I could discard it after one play session. At that stage I could hand large pieces of implementation to the models and explore alternatives quickly. In the Android project, the same degree of trust began to slow the work down. The codebase grew, dependencies formed between components, and fixing one problem could violate an assumption on which another part of the application relied.

> **“The larger the project became, the less work I could hand over to AI without question.”**

As the project evolved, AI's role shifted from generating entire implementations to providing engineering support. It became more useful to discuss several possible solutions with a model, test hypotheses, locate the source of a discrepancy, and prepare small, controlled changes. Every significant step still required tests and manual verification, and the final decision remained with the person who understood the product and was responsible for the result.

Keeping the documentation in sync with the code also became essential. If I corrected behaviour by hand, preserving the change only in the repository was not enough. The new rule or architectural decision also had to appear in the documents the models received. Without that, the human and the AI would begin working from different versions of the project within a few iterations. A model would reproduce an old constraint or try to restore a discarded decision, wasting time on contradictions that an up-to-date context could have prevented.

This experience did not show that AI makes game development easy or removes the need to learn a new stack. Its value lay elsewhere: these tools got me to the first testable version before the cost of getting started could make me abandon the idea.

After the successful HTML prototype, the problem changed. Doubts about the mechanic gave way to the ordinary questions of product development: how to maintain the code, where to draw state boundaries, how to verify changes, and how to preserve knowledge about past decisions. That is how an experiment with several models and self-contained HTML files gradually became an Android project—and a memory of a nameless Flash game became Synaptic Front.

> **Try Synaptic Front**
>
> The game is already available on [Google Play](https://play.google.com/store/apps/details?id=me.vasylchenko.synapticfront). You can see what the idea in this story became and play it for yourself.
>
> *To be continued.*
