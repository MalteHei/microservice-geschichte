# Microservices - Teil 4: Frontend(s) in einer Microservice-Umgebung

Im vorherigen Teil dieser Artikel-Serie wurde ein kleiner Einblick in die Welt der Kubernetes Ingress Controller gewährt. Dabei habe ich drei verschiedene Controller basierend auf meinen eigenen Erfahrungen verglichen.

In diesem Artikel wird erläutert, welche Pattern existieren, um Frontends für Microservice-Architekturen zu erstellen.

## Frontend-Monolithen

Es scheint offensichtlich, eine einzige (monolithische) Frontend-Anwendung zu entwickeln, in welcher die APIs der verschiedenen Microservices aufgerufen und die Ergebnisse daraus aggregiert werden. Dabei ist die Entwicklung des Frontends unabhängig von den Microservices im Backend; das Frontend kommuniziert lediglich mit den Endpunkten, welche von den Microservices spezifiziert wurden. Diese Endpunkte werden in Form von versionierten APIs von den Backend-Teams bereitgestellt, während ein eigenes Frontend-Team dafür zuständig ist, diese zu konsumieren.
Hierdurch entsteht eine _enge Kopplung_, da Features erst am User ankommen, wenn vom Frontend eine entsprechende neue Version erscheint. Jedoch zur Erinnerung: Ziel der Microservice-Architektur war es, solche Kopplungen zu lösen, um z. B. hohe Grade der Modularität und Skalierbarkeit zu erreichen \[[1]\].

Das bedeutet allerdings nicht, dass die Entwicklung eines Frontend-Monolithen keine gute Idee sein kan. Wenn beispielsweise die Anzahl an Teams gering ist oder es nur wenige Funktionalitäten gibt, dann kann eine solche Anwendung durchaus die beste Wahl sein. Wenn es jedoch viele Teams gibt oder die Spanne an Funktionalitäten sehr groß ist, sollte man eher auf die _Microfrontend_-Architektur zurückgreifen \[[1]\].

## Microfrontends

Die Idee hinter Microfrontends ist dieselbe wie bei Microservices: Eine enge Kopplung von Komponenten soll eliminiert werden, damit diese jeweils unabhängig voneinander entwickelt und ausgeliefert werden können. Dabei übernimmt jedes Microservice-Team nicht nur die Verantwortung für den jeweiligen Microservices, sondern auch für das dazugehörige Frontend (siehe Abbildung 1) \[[1]\] \[[2]\].

<center>

  ![Frontends sind den Microservice-Teams zugeteilt][img:micro-frontends]

  _Abbildung 1: Microfrontends in einer Teamstruktur_ \[[1]\]
</center>

Durch den Einsatz von Microfrontends erhält man die selben Vorteile wie bei Microservices: Frontend-Komponenten sind durch kleine Codebases besser **wartbar**, können durch die lose Kopplung **skaliert** und bei Bedarf einfach **neu geschrieben**/**upgedatet**/**upgegradet** werden \[[2]\].

Im Folgenden werden einige Pattern zur Integration der Microfrontend-Architektur vorgestellt.

### Build-time Integration

Bei diesem Pattern wird jedes Frontend als eigenes [Package bzw. Modul][site:npm-packages] bereitgestellt. Zusätzlich wird eine Container-Anwendung erstellt, in der die Frontend-Packages der verwendeten Microservices als Abhängigkeiten aggregiert und zur Build-Zeit kompiliert werden.
Dabei ist eben dieser letzte Schritt der der Nachteil dieses Patterns: Bei der kleinsten Änderung an einem der Frontend-Packages muss die komplette Container-App, inklusive aller anderen Packages, neu gebaut und ausgerollt werden. Dadurch wird die lose Kopplung aufgehoben, welche überhaupt der Grund zur Einführung einer Microfrontend-Architektur war, weshalb man das Pattern der Build-time Integration **vermeiden** sollte \[[1]\] \[[2]\].

### Runtime Integration

Das Runtime Integration-Pattern ähnelt der Build-time Integration, jedoch werden die Abhängigkeiten in der Container-Anwendung hier (meist clientseitig) zur Laufzeit bezogen, wodurch die Komponenten weiterhin lose gekoppelt bleiben \[[1]\] \[[2]\].

Folgende Ansätze können u. a. zur Implementation dieses Patterns verwendet werden:

- **Web Components**:

  [Web Components][site:web-components] basieren auf dem Webstandard [Custom Elements][site:custom-elements], welcher die Definition individueller HTML-Elemente ermöglicht. Durch das Beziehen der Microfrontends werden demnach eigene Komponenten definiert, welche anschließend von der Container-App auf der Seite angezeigt werden können \[[1]\] \[[2]\].

- **JavaScript**:

  Bei diesem Ansatz definieren die Microfrontends keine Komponenten, sondern globale Funktionen (bspw. `window.renderUserProfile`). Diese Funktionen sind für die Erstellung sowie Darstellung der UI-Elemente zuständig und können basierend auf dem angefordertem Pfad aufgerufen werden \[[1]\] \[[2]\].

- **Single SPA**:

  Eine Single Page Application (SPA) ist eine Webanwendung, die aus einer einzigen HTML-Seite besteht. Ressourcen und Funktionalitäten werden nachgeladen, sobald diese benötigt werden. Das [_single-spa_][site:single-spa]-Framework kann verwendet werden, um eine Microfrontend-Umgebung aufzubauen, die aus SPAs besteht. Dabei werden neben Angular und React noch [weitere SPA-Frameworks unterstützt][site:single-spa:ecosystem] \[[1]\].

## Fazit

## Referenzen

\[1\]: A. Fleischer, H. Schröder (2021), Frontends with Microservices – Why the choice of frontend architecture also impacts team collaboration, <https://www.otto.de/jobs/technology/techblog/artikel/frontends-with-microservices.php>

\[2\]: C. Jackson (2019), Micro Frontends, <https://martinfowler.com/articles/micro-frontends.html>

<!----------------------------->
<!-- Referenzen für Markdown -->
[1]: https://www.otto.de/jobs/technology/techblog/artikel/frontends-with-microservices.php
[2]: https://martinfowler.com/articles/micro-frontends.html
[img:micro-frontends]: img/micro-frontends.png
[site:npm-packages]: https://docs.npmjs.com/about-packages-and-modules
[site:web-components]: https://developer.mozilla.org/en-US/docs/Web/Web_Components
[site:custom-elements]: https://html.spec.whatwg.org/multipage/custom-elements.html
[site:single-spa]: https://github.com/single-spa/single-spa
[site:single-spa:ecosystem]: https://single-spa.js.org/docs/ecosystem/
