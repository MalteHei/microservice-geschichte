# Microservices - Teil 2: Entwicklung von Microservices

Im ersten Teil dieser Artikel-Serie wurde die Geschichte der Microservice-Architektur dargestellt. Dazu wurde die Entstehung dieses Konzeptes sowie dadurch gelöste Probleme aber auch daraus entstandenen Kompromisse vorgestellt. Außerdem wurde anhand einer Umfrage aus 2020 festgestellt, dass Node.js die populärste Technologie zur Entwicklung von Microservices war. Die Silbermedaille ging dabei an Java, was vermutlich am Enterprise-Bereich liegt: Java ist vor allem dank eines großen Ökosystems und in Kombination mit dem Spring Framework eine optimale Technologie zur Lösung vieler komplexer Probleme, welche oft in Enterprise-Umgebungen vorhanden sind \[[4]\]\[[5]\].

In diesem Artikel werden daher verschiedene Methoden erläutert, welche die Orchestrierung sowie Entwicklung von Microservices mit Spring Boot erleichtern, und hinsichtlich Service Discovery, Load Balancing, Resilienz und Routing verglichen.

## Spring Cloud Netflix (Netflix-Stack)

Netflix bietet auf [GitHub][site:netflix-oss] eine Vielzahl an Open-Source Technologien an, die vom Unternehmen selbst eingesetzt werden, um dessen Streaming-Dienste bereitzustellen. Das Spring Team stellt mit **[Spring Cloud Netflix][site:spring-cloud-netflix]** ein häufig eingesetztes Framework zur Entwicklung von Microservices in Java zur Verfügung, welches Entwicklern erlaubt, diese bewährten Komponenten ohne großen Aufwand zu verwenden. Im Folgenden wird erklärt, welche Komponenten dieses Framework mit sich bringt und welche Funktionen diese tragen \[[1]\].

### Eureka

Damit Microservices miteinander kommunizieren können, muss bekannt sein, welcher Service unter welcher Netzwerkadresse erreichbar ist. Alternativ zum manuellen Eintragen aller IP-Adressen in eine Config-Datei (diese IPs könnten sich ständig ändern), kann ein [Eureka][site:eureka] Server eingesetzt werden, welcher die **Service Discovery** (= zentralisierte Verwaltung der Adressen aller Microservices) übernimmt. Eureka Service Clients können sich dann per HTTP-Request am Eureka Server registrieren und Application Clients können wiederum die Adressen dieser registrierten Services am Eureka Server abfragen.

> Aufgrund einiger Limitationen von Eureka wurde entschlossen, ein neues Major Release (Eureka 2.0) mit diversen Verbesserungen wie bspw. interessenbasierten Subscriptions der Service Registry oder einem informativeren Dashboard zu entwickeln. Die Entwicklung von Eureka 2.0 wurde jedoch eingestellt, da parallele Open-Source Projekte nie die benötigte Reife erreichten \[[2]\]. Version 1 erhält weiterhin Updates: <https://github.com/Netflix/eureka/releases>.

### Hystrix

Der Ausfall eines Services in einem verteilten System ist in den meisten Fällen unumgänglich. Da es in Microservice-Architekturen nicht ungewöhnlich ist, dass die Services sich gegenseitig aufrufen, sollte ein gewisser Grad an **Resilienz** in Form von Ausfallsicherheit sichergestellt werden. [Hystrix][site:hystrix] ist Netflix' Implementierung des _Circuit Breaker_-Patterns (temporäre Unterbindung aller Aufrufe eines Services im Fehlerfall).

### Ribbon

Ein Vorteil der Microservice-Architektur ist, dass jeder Service unabhängig skaliert werden kann, sodass häufig mehrere Instanzen eines Services im System existieren. Damit die Anfragen auf einen Service fair über alle Instanzen verteilt werden, kann ein Load Balancer eingesetzt werden, welcher eben dies nach einem vordefinierten Verfahren regelt. [Ribbon][site:ribbon] ist Netflix' Implementierung eines **clientseitigen Load Balancers**.

### Zuul

[Zuul][site:zuul] fungiert in der Rolle eines **Gateway Proxy** als Eintrittspunkt in ein verteiltes System, wodurch der interne Aufbau des jeweiligen Systems nach außen hin verborgen werden kann. Dabei können Anfragen dynamisch an verschiedene Backend-Cluster geleitet (**Routing**), parallel **serverseitiges Load Balancing** betrieben und außerdem die **Resilienz** erhöht werden.

## Spring Cloud Kubernetes

**[Spring Cloud Kubernetes][site:spring-cloud-k8s]** kann als Dependency zu einem Spring Boot-Projekt hinzugefügt werden, wodurch Implementationen diverser Interfaces bereitgestellt werden. Diese sollen die Entwicklung von Spring Cloud-Anwendungen in einer Kubernetes Umgebung vereinfachen.

Um alle Funktionalitäten von Spring Cloud Kubernetes in einem Microservice nutzen zu können, muss folgender Eintrag in der `pom.xml` des jeweiligen Services ergänzt werden:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-kubernetes-all</artifactId>
  <version>1.1.10.RELEASE</version>
</dependency>
```

> Version `1.1.10` ist aktuell die neueste Version dieser Abhängigkeit und kompatibel mit Spring Boot `2.3.X`.

### Service Discovery (clientseitig)

Um in Erfahrung zu bringen, welche Services in Kubernetes zur Verfügung stehen, kann ein `DiscoveryClient` verwendet werden.
Dazu muss die Annotation `@EnableDiscoveryClient` in der Spring Boot Application-Klasse ergänzt werden:

```java
@SpringBootApplication
@EnableDiscoveryClient
public class MicroserviceA {
  public static void main(String[] args) {
    SpringApplication.run(MicroserviceA.class, args);
  }
}
```

Daraufhin kann per Dependency Injection ein `DiscoveryClient` erstellt werden, welcher die Kubernetes API auf Services abfragt:

```java
@Autowired
private DiscoveryClient discoveryClient;

// List<String> all_services = discoveryClient.getServices();
// List<ServiceInstance> ms_b_instances = discoveryClient.getInstances("myapp-microservice-b");
// ...
```

> Die serverseitige Service Discovery, welche nativ in Kubernetes vorhanden ist, kann natürlich unabhängig vom `DiscoveryClient` verwendet werden.

Das Besondere an dieser Implementation ist, dass Services in allen Kubernetes-Namespaces gefunden werden können:

```yaml
spring:
  cloud:
    kubernetes:
      discovery:
        all-namespaces: true
```

### Load Balancing (clientseitig)

Neben der clientseitigen Service Discovery bringt Spring Cloud Kubernetes ebenfalls einen clientseitigen Load Balancer mit sich.
Dieser kann aktiviert werden, indem bspw. ein `RestTemplate`-Bean mit der Annotation `@LoadBalanced` versehen wird \[[3]\]:

```java
@Bean
@LoadBalanced
RestTemplate restTemplate() {
  return new RestTemplate();
}
```

Ähnlich wie bei der Service Discovery kann der Load Balancer auch über verschiedene Namespaces hinweg fungieren:

```yaml
spring:
  cloud:
    kubernetes:
      discovery:
        all-namespaces: true
```

### Resilienz

Spring Cloud Kubernetes [schlägt vor][site:spring-cloud-k8s-resilience], **Hystrix** aus dem Netflix-Stack als Implementation des _Circuit-Breaker_-Patterns sowie zur Angabe von Fallback-Methoden zu verwenden.

### Routing

Für das Routing muss auf **Zuul**, ebenfalls eine Komponente des Netflix-Stacks, zurückgegriffen werden \[[3]\].

## Kubernetes nativ

Kubernetes bietet eine All-in-One-Lösung als Tool zur Orchestrierung von Software-Containern. Doch kann Kubernetes nativ wirklich alle Komponenten des bewährten Netflix-Stacks ablösen?

### Resilienz via Replica-Sets und Probes

Falls ein Pod ausfällt, wird dieser von Kubernetes automatisch neu gestartet. Damit der Service währenddessen weiterhin verfügbar ist, können mehrere Instanzen des Pods parallel ausgeführt werden. Um die Resilienz eines Kubernetes Clusters auf diese Weise zu erhöhen, können relativ einfach sogenannte `ReplicaSet`s eingesetzt werden. Ein `ReplicaSet` ist ein Kubernetes-Objekt, das aus mehreren Instanzen bzw. Replicas eines Pods besteht:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-component-a
spec:
  replicas: 10
  # ...
```

Zusätzlich sollten Liveness & Readiness Probes für jeden Container definiert werden. Dabei stellt Kubernetes über Liveness Probes fest, ob ein Container neu gestartet werden soll. Über Readiness Probes wird festgestellt, ob ein Service zu dem jeweiligen Zeitpunkt Anfragen verarbeiten kann.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-component-a
spec:
  replicas: 10
  # ...
  containers:
  - # name: ...
    # image: ...
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

### Routing innerhalb eines Clusters via Services

Routing innerhalb des Clusters kann mithilfe von Service-Ressourcen erreicht werden. Die Adresse eines Services wird dabei über das Attribut `metadata.name` definiert:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-a-service
spec:
  type: ClusterIP # Defaultwert
  selector:
    app: myapp-component-a
  ports:
  - port: 8080
    targetPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-b-service
spec:
  selector:
    app: myapp-component-b
  ports:
  - port: 8080
    targetPort: 8080
```

Ein Container innerhalb von `myapp-component-a` könnte so mit `myapp-component-b` kommunizieren, indem Anfragen an `http://myapp-b-service:8080/` gesendet werden.

### Load Balancing via Services

Load Balancing kann in Kubernetes u. a. serverseitig durch Services realisiert werden. Wenn bspw. ein `ReplicaSet` existiert...

```yaml
# deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 10
  # ...
```

... kann ein Service für dieses Deployment erstellt werden:

```yaml
# service.yaml

apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  # ports: ...
```

Der `myapp-service` verteilt dann automatisch Anfragen für die angegebenen Ports auf die Instanzen von `myapp` im ReplicaSet. Diese Funktionalität macht eine **clientseitige Service Discovery obsolet**, da Clients lediglich die Namen der Services kennen müssen, um auf diese zuzugreifen.
Durch die Angabe von `type: LoadBalancer` wird außerdem ermöglicht, dass die im Service angegebenen Ports außerhalb des Kubernetes Clusters angesprochen werden können.

### Routing außerhalb eines Clusters via Ingress

Wenn man nicht möchte, dass Container außerhalb des Clusters _direkt_ angesprochen werden können, so können API Gateways mithilfe von [Ingress][site:k8s-ingress] erstellt werden.

Wenn z. B. folgende Services existieren:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-frontend-service
spec:
  # selector: ...
  # ports: ...
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-microservice-a
spec:
  # ...
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-microservice-b
spec:
  # ...
```

... kann eine Ingress-Ressource erstellt werden, welche Regeln für alle 3 Services enthält:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  ingressClassName: <ingress-class> # z. B. `nginx`
  rules:
  - host: localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-frontend-service
            port:
              number: 80
      - path: /api/ms-a
        pathType: Prefix
        backend:
          service:
            name: myapp-microservice-a
            port:
              number: 80
      - path: /api/ms-b
        pathType: Prefix
        backend:
          service:
            name: myapp-microservice-b
            port:
              number: 80
```

Nach dem Anwenden dieser Konfigurationen könnte man über...

- `localhost:80/` auf das Frontend,
- `localhost:80/api/ms-a` auf Microservice A und
- `localhost:80/api/ms-b` auf Microservice B

zugreifen.

> Es sollte beachtet werden, dass zusätzlich zum Ingress-Objekt zwingend ein **Ingress Controller** (z. B. [von NGINX][site:nginx-ingress-controller]) benötigt wird. Dieser Controller interpretiert die Regeln der Ingress-Ressource und führt basierend darauf das Routing durch. Mehr zu Ingress Controllern jedoch im nächsten Teil dieser Artikel-Serie.

## Fazit

In diesem Artikel wurden der Netflix-Stack (Spring Cloud Netflix) mit Spring Cloud Kubernetes sowie nativen Kubernetes-Funktionalitäten hinsichtlich Service Discovery, Load Balancing, Resilienz und Routing verglichen.

Dabei stellte sich heraus, dass Spring Cloud Kubernetes eine essenzielle Erweiterung zu Sprig Cloud Netflix ist, wenn man Kubernetes zur Orchestrierung seiner Microservices verwenden möchte. Hierdurch kann nämlich beispielsweise auf einen Eureka Server als zusätzliche Komponente im System verzichtet werden. Jedoch kann Spring Cloud Netflix nicht gänzlich durch Spring Cloud Kubernetes ersetzt werden, da für Resilienz und Routing nach wie vor auf den Netflix-Stack zurückgegriffen werden muss.

Dahingegen konnte bei Kubernetes festgestellt werden, dass die hier thematisierten Funktionalitäten nativ verfügbar sind und ohne großen Aufwand implementiert werden können. Somit kann man sich den Aufwand sparen, der mit der Entwicklung sowie Wartung von extra Services (Eureka Server usw.) verknüpft ist.

Im nächsten Teil dieser Artikel-Serie ist ein kleiner Exkurs in Kubernetes Ingress Controller und beinhaltet meine persönlichen Erfahrungen mit drei verschiedenen Projekten.

## Referenzen

\[1\]: Carsten Pelka, Michael Plöd, Microservices à la Netflix - Die Bestandteile von Spring Cloud Netflix, <https://www.innoq.com/de/articles/2016/12/microservices-a-la-netflix/>

\[2\]: Eureka 2.0 Motivations, <https://github.com/Netflix/eureka/wiki/Eureka-2.0-Motivations>

\[3\]: Piotr Minkowski, Spring Cloud Kubernetes Load Balancer Guide, <https://piotrminkowski.com/2020/09/10/spring-cloud-kubernetes-load-balancer-guide/>

\[4\]: State of Microservices 2020 Report, <https://tsh.io/state-of-microservices/#ebook>

\[5\]: Rhuan Rocha, Why Java is so hot right now, <https://developers.redhat.com/blog/2019/09/05/why-java-is-so-hot-right-now>

<!----------------------------->
<!-- Referenzen für Markdown -->
[1]: https://www.innoq.com/de/articles/2016/12/microservices-a-la-netflix/
[2]: https://github.com/Netflix/eureka/wiki/Eureka-2.0-Motivations
[3]: https://piotrminkowski.com/2020/09/10/spring-cloud-kubernetes-load-balancer-guide/
[4]: https://tsh.io/state-of-microservices/#ebook
[5]: https://developers.redhat.com/blog/2019/09/05/why-java-is-so-hot-right-now
<!-- Seiten & Bilder -->
[site:netflix-oss]: https://netflix.github.io/
[site:spring-cloud-netflix]: https://spring.io/projects/spring-cloud-netflix
[site:eureka]: https://github.com/Netflix/eureka
[site:hystrix]: https://github.com/Netflix/hystrix
[site:ribbon]: https://github.com/Netflix/ribbon
[site:zuul]: https://github.com/Netflix/zuul
[site:k8s-ingress]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[site:nginx-ingress-controller]: https://kubernetes.github.io/ingress-nginx/deploy/
[site:spring-cloud-k8s]: https://docs.spring.io/spring-cloud-kubernetes/docs/current/reference/html/
[site:spring-cloud-k8s-resilience]: https://docs.spring.io/spring-cloud-kubernetes/docs/current/reference/html/#kubernetes-native-service-discovery
