# Microservices - Teil 1: Die Geschichte von Microservices

In einer Microservice-Architektur wird eine große Anwendung in alleinstehende Komponenten bzw. Services aufgeteilt. Diese sogenannten Microservices können dann unabhängig voneinander entwickelt und ausgeführt werden und kommunizieren untereinander meist über HTTP.

## Entstehung

Dieser Architekturstil ist keine Neuheit: Die Grundidee dahinter bestand schon seit den 2000ern \[[1]\], allerdings wurde der Begriff _Microservice_ erstmals 2011 auf einem Workshop für Softwarearchitektur in Venedig verwendet \[[2]\]. Anschließend wurde dieser Stil zu großen Teilen von **James Lewis** und **Martin Fowler** populär gemacht \[[3]\]. Das Ziel war stets eine Softwarearchitektur, welche die Anforderungen an moderne Systeme möglichst gut erfüllen kann.

## Gelöste Probleme

<center>

  ![Monolithen vereinen Funktionalitäten; Microservices verteilen Funktionselemente in eigene Services][img:monolith]

  _Abbildung 1: Monolith vs Microservice_ \[[4]\]
</center>

Microservices sind der Gegenentwurf zu sogenannten Monolithen (= große Systeme, die alle Funktionen in einem einzigen Prozess vereinen, Vgl. Abb. 1). Diese Systeme können genauso erfolgreich sein wie Microservices, allerdings bringen sie auch einige Probleme mit sich, die schnell für Frustration sorgen können. Ein großes Problem von Monolithen ist, dass bei der kleinsten Änderung das _komplette System_ neu gebaut und deployed werden muss. Bei einer Microservice-Architektur hingegen müsste dies lediglich bei dem aktualisierten Service geschehen. Durch die enge Kopplung eines Monolithen ist es außerdem nicht leicht, eine gute **modulare Struktur** auf längere Zeit beizubehalten. Des Weiteren wird die **Skalierbarkeit** beeinträchtigt, da wenn eine kleine Komponente mehr Ressourcen benötigt, alle anderen Komponenten des Systems ebenfalls skaliert werden müssten \[[4]\]. Ein weiteres Problem von Monolithen ist das **Debugging von Fehlern**, die durch Änderungen am Code eingeführt wurden. Fehler in solchen Systemen legen oft die gesamte Anwendung lahm und der Fehlerursprung kann nicht immer eindeutig identifiziert werden (vor allem, wenn mehrere Entwickler gleichzeitig Änderungen am Code vornehmen). Dies war auch der Grund, warum unter anderem Netflix auf eine Microservice-Architektur umgestiegen ist \[[5]\].

## Kompromisse

Wenn die Microservice-Architektur nur Vorteile mit sich bringen würde, würden wahrscheinlich alle Systeme nach diesem Stil entwickelt werden. Da dies nicht der Fall ist, bringen Microservices offensichtlich auch einige Nachteile und Kompromisse mit sich.
Durch die Interprozesskommunikation (via HTTP) erhöhen sich beispielsweise die **Antwortzeiten** sowie die Anzahl von **Timeouts**.
Zusätzlich muss darauf geachtet werden, dass ein gewisser Grad an **Resilienz** erreicht wird. Das bedeutet, dass ein Microservice nicht durch den Ausfall eines anderen Services blockiert werden darf. Dies hätte nämlich einen Domino-Effekt zufolge, wodurch ein komplettes System lahmgelegt werden könnte. Um dem entgegen zu wirken und einen gewissen Grad an Ausfallsicherheit herzustellen, gibt es diverse Pattern. Darunter fallen beispielsweise das _Timeout_- (Begrenzung von Service-Durchlaufzeiten), _Circuit Breaker_- (temporäre Unterbindung aller Aufrufe eines Services im Fehlerfall) oder _Retry_-Pattern (autom. Wiederanlauf fehlgeschl. Aufrufe) \[[10]\].
Da ein Microservice dennoch jederzeit ausfallen kann, sollten außerdem Maßnahmen bezüglich **Monitoring und Logging** ergriffen werden. Metriken, die häufig in der Praxis überwacht werden, sind z. B. Latenz, Durchsatz oder Up-/Down-Status.
Des Weiteren besteht bei Microservices ein relativ **hoher Initialaufwand**, da die Infrastruktur für Builds, Tests, Deployment etc. (CI/CD) für jeden Service einzeln aufgesetzt werden muss \[[3]\]\[[7]\].

### Choreographie vs. Orchestrierung

Im Microservice-Kontext wird zwischen zwei Pattern, welche festlegen, wie die Services aufgerufen werden, unterschieden. Zum einen gibt es die **Orchestrierung**, welche man sich analog zu einem Orchester vorstellen kann: Jeder Musikant im Orchester weiß, wie sein Instrument zu spielen ist - der Beitrag zum Gesamtstück wird jedoch von einem Dirigenten geleitet. In einer Microservice-Architektur würde der Dirigent durch einen separaten Service dargestellt werden, welcher die anderen Services orchestriert, um einen bestimmten Prozess (z. B. Registration eines Users) durchzuführen.
Dem gegenüber steht das **Choreographie**-Pattern, welches analog mit bspw. einer Tanzaufführung verglichen werden kann: Alle Tänzer wissen unabhängig voneinander, was ihr Beitrag zum Ganzen ist. Die Choreographie ist Event-basiert, d. h. dass Services einen Event-Bus abonnieren können, auf dem Events anderer Services (z. B. _UserRegistered_) veröffentlicht werden. Hierzu können Message Broker wie [Apache Kafka][site:kafka] oder [RabbitMQ][site:rabbit] eingesetzt werden \[[8]\]\[[9]\].

## Aktueller Stand

Im Jahr 2020 hat [The Software House][site:tsh] im Rahmen der State of Microservices 2020 \[[6]\] 669 Entwickler befragt, wie sie mit Microservices arbeiten.

### Erfahrungen mit Microservices

Daraus ging hervor, dass die meisten Entwickler sehr gute Erfahrungen mit der **Skalierbarkeit** sowie der **Performance** von Microservices gemacht haben. Lediglich bei der Wartbarkeit und dem Debugging konnten einige Befragte noch keine guten Erfahrungen aufweisen.

### Cloud-Computing

Provider von Cloud-Computing spielen eine große Rolle in Verbindung mit Microservices: 50% aller Befragten gaben an, ihre Systeme auf [Amazon Web Services][site:aws] zu deployen. Auf eigenen Servern würden Microservices von ~35% der Befragten deployed. [Azure][site:azure] und [Google Cloud Platform][site:gcp] fänden lediglich bei jeweils 17% Verwendung.
Dabei verwendet ziemlich genau die Hälfte der Befragten **Serverless-Architekturen**; und darunter hauptsächlich AWS Lambda.

Mit diesem großen (und vermutlich weiter steigenden) Anteil an Cloud-Computing im Microservices-Kontext scheint ein Kontakt mit Software zur Containervirtualisierung schier unvermeidbar. **Docker** zählt aktuell als bekannteste Lösung dafür, Systeme in Form von Containern bereitzustellen. In Verbindung mit Docker wird oft **Kubernetes** als Tool zur Verwaltung jener Containern erwähnt. Kubernetes erlaubt das automatische Starten/Stoppen, Überwachen und Aktualisieren von Containern und benötigt dafür lediglich eine Beschreibung des gewünschten Zustandes in Form einer Config-Datei (z. B. welche Container in welcher Anzahl benötigt werden) \[[11]\].

### Technologien

Für lange Zeit galt Java als Go-To-Technologie, wenn es um die Entwicklung von Microservices ging. Die Ergebnisse dieser Umfrage zeigen jedoch, dass die Popularität von [Node.js][site:nodejs] in Form von **JavaScript und [TypeScript][site:ts]** auf Microservice-Architekturen übergesprungen ist: 65,3% aller Befragten gaben an, hauptsächlich JS/TS in Projekten zu verwenden und nur 26,3% würden Java als Haupt-Technologie einsetzen. Fast genau so viele Befragte (25,6%) gaben an, _ausschließlich_ JS/TS in Microservices zu nutzen.

In der Java-Welt hat sich jedoch einiges getan: Neben dem relativ alten jedoch allzeit beliebtem Framework **Spring** entsteht ein immer größer werdender Hype um neue Java-Frameworks wie **[Micronaut][site:micronaut]** (2018) oder **[Quarkus][site:quarkus]** (2019) \[[12]\]. Diese werben mit schnellen Coding-Workflows, minimalen Startzeiten, schneller und einfacher Entwicklung von Tests sowie nativem Support für Cloud-Technologien (z. B. Kubernetes).

### DevOps

Im DevOps-Bereich wurde festgestellt, dass bei einem Großteil der Microservices (86,7%) **Continuous Integration** eingesetzt werde, um bestimmte Aspekte dieser zu automatisieren. Dabei sticht **GitLab CI** als die CI-Lösung heraus, welche von dem meisten Entwicklern (25%) präferiert wird. Darauf folgen u. a. Jenkins, GitHub Actions, Circle CI und Bitbucket Pipelines, welche mit jeweils ~13% ähnlich beliebt unter Entwicklern sind.

### Prognosen

Mit Blick auf die Zukunft sind sich 85,5% der Befragten sicher, dass die Microservice-Architektur der Industriestandard entweder für Backends (36,2%) oder komplexe Systeme (49,3%) sein wird.

## Fazit

In diesem Artikel wurde die Geschichte von Microservices thematisiert. Dabei wurde festgestellt, dass das Konzept von verteilten Systemen bereits seit mehreren Jahrzehnten besteht, allerdings erst seit ~10 Jahren benannt und populär gemacht wird. Außerdem wurde aufgeführt, welche Probleme durch eine Microservice-Architektur gelöst werden, aber auch welche Kompromisse dadurch entstehen. Schließlich wurde auf Basis einer Umfrage der aktuelle Stand von Microservices bezüglich verwendeter Technologien usw. dargelegt. Hier stellte sich heraus, dass Cloud-Computing sowie Node.js und Java als Technologien in Kombination mit Microservices eine große Rolle spielen.

Im nächsten Teil dieser Artikel-Serie werden verschiedene Methoden/Frameworks vorgestellt, welche die Orchestrierung sowie Entwicklung von Microservices in Java mit Spring erleichtern. Dabei werden der Netflix-Stack in Form von Spring Cloud Netflix, native Kubernetes-Funktionalitäten und Spring Cloud Kubernetes hinsichtlich Service Discovery, Load Balancing, Resilienz und Routing verglichen.

## Referenzen

\[1\]: That Tech Show, Episode 14 - The Grandfather of Microservices, Fred George, <https://play.acast.com/s/that-tech-show/fred-george-the-grandfather-of-microservices>

\[2\]: Dragoni N. et al. (2017) Microservices: Yesterday, Today, and Tomorrow. In: Mazzara M., Meyer B. (eds) Present and Ulterior Software Engineering. Springer, Cham. <https://doi.org/10.1007/978-3-319-67425-4_12>

\[3\]: Martin Fowler, Microservices Guide, <https://martinfowler.com/microservices/>

\[4\]: Martin Fowler, Microservices, <https://martinfowler.com/articles/microservices.html>

\[5\]: Josh Evans, Mastering Chaos - A Netflix Guide to Microservices, <https://youtu.be/CZ3wIuvmHeM?t=385>

\[6\]: State of Microservices 2020 Report, <https://tsh.io/state-of-microservices/#ebook>

\[7\]: Eberhard Wolff, Warum Microservices scheitern, <https://www.innoq.com/de/articles/2019/10/warum-microservices-scheitern>

\[8\]: Ethan Garofolo, Choreography vs Orchestration in long running asynchronous processes, <https://youtu.be/ofJd3AZIfto>

\[9\]: Bernd Rücker, Balancing Choreography and Orchestration, <https://youtu.be/zt9DFMkjkEA>

\[10\]: Nygard, M. T. (2018). Release It! Design and Deploy Production-Ready Software. USA: Pragmatic Bookshelf. <https://pragprog.com/titles/mnee2/release-it-second-edition/>

\[11\]: Manuel Reinfurt, Kubernetes – Alles, was Sie darüber wissen müssen. <https://www.incloud.de/magazin/kubernetes>

\[12\]: Hartmut Schlosser, Java-Trends: Top 10 der Frameworks im Jahr 2020, <https://entwickler.de/java/java-trends-top-10-der-frameworks-im-jahr-2020/>

<!----------------------------->
<!-- Referenzen für Markdown -->
[1]: https://play.acast.com/s/that-tech-show/fred-george-the-grandfather-of-microservices
[2]: https://doi.org/10.1007/978-3-319-67425-4_12
[3]: https://martinfowler.com/microservices/
[4]: https://martinfowler.com/articles/microservices.html
[5]: https://youtu.be/CZ3wIuvmHeM?t=385
[6]: https://tsh.io/state-of-microservices/#ebook
[7]: https://www.innoq.com/de/articles/2019/10/warum-microservices-scheitern
[8]: https://youtu.be/ofJd3AZIfto
[9]: https://youtu.be/zt9DFMkjkEA
[10]: https://pragprog.com/titles/mnee2/release-it-second-edition/
[11]: https://www.incloud.de/magazin/kubernetes
[12]: https://entwickler.de/java/java-trends-top-10-der-frameworks-im-jahr-2020
<!-- Seiten & Bilder -->
[img:monolith]: img/monolith-vs-microservice.png
[site:aws]: https://aws.amazon.com/
[site:azure]: https://azure.microsoft.com/
[site:gcp]: https://cloud.google.com/
[site:tsh]: https://tsh.io/
[site:nodejs]: https://nodejs.org/
[site:ts]: https://www.typescriptlang.org/
[site:kafka]: https://kafka.apache.org/
[site:rabbit]: https://www.rabbitmq.com/
[site:quarkus]: https://quarkus.io/
[site:micronaut]: https://micronaut.io/
