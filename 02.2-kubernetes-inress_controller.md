# Kubernetes Ingress Controller

Im Rahmen dieser Artikel-Serie habe ich mich viel mit [Kubernetes Ingress][ingress] beschäftigt.
Dabei fiel mir jedoch die Wahl eines Ingress Controllers aufgrund der schieren Anzahl an [verfügbaren Optionen][controller:overview] schwer.

In diesem Artikel werde ich daher meine _persönlichen Erfahrungen_ mit einer willkürlichen Auswahl an Ingress Controllern darlegen (in der Hoffnung, die Wahl eines geeigneten Controllers für Ihr Projekt zu erleichtern).

> Für diesen Artikel wird ein grundlegendes Verständnis von [Kubernetes Ingress][ingress] vorausgesetzt.

## NGINX Ingress Controller

Der [NGINX Ingress Controller][nginx:github] war aufgrund der [offiziellen Unterstützung vom Kubernetes Projekt][nginx:official-support] der erste Controller, den ich ausprobiert habe.

### Installation

Die [Installation][nginx:install] via Helm ist dabei sehr komfortabel (alternativ wird YAML-Manifest online bereitgestellt):

```bash
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

Anschließend kann der Controller verwendet werden, indem der `ingressClassName` für eine Ingress Ressource gesetzt wird:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-nginx-ingress
spec:
  ingressClassName: nginx
  # ...
```

### Features

Zu den wichtigsten [Features dieses Controllers][nginx:annotations] zählen:

- Routing anhand von Pfaden (mit RegEx) und Rewriting dieser (unterstützt RegEx-Gruppen aus Pfad)
- Authentifizierung (BasicAuth oder OAuth über externe Anbieter)
- Canary Release
- Rate Limiting
- CORS

Diese Features können über Annotationen an der Ingress Ressource konfiguriert werden:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
    # ...
spec:
  ingressClassName: nginx
  # ...
```

### Nachteile

Ein Nachteil des NGINX Ingress Controllers ist, dass die Methoden der Authentifizierung stark limitiert sind.
Außerdem können Routing-Regeln nur aus einer Kombination von Host + Pfad bestehen, was eine deutliche Einschränkung gegenüber alternativen Controllern ist.

> Das Routing anhand von Headern ist bei diesem Controller ebenfalls möglich (allerdings nur über Umwege, z. B. [Canary][nginx:canary]). Dadurch könnte man beispielsweise durch die Angabe des Headers `X-Branch: dev` auf die Dev-Version des angeforderten Services zugreifen.

## Kong Ingress Controller (KIC)

Der [_Kong Ingress Controller for Kubernetes_][kong:github] (KIC) basiert auf dem beliebtem [Kong API Gateway][kong:kong] und versucht, mit vielseitigen Features punkten zu können.

### Installation

Auch dieser Ingress Controller kann über ein [Helm Chart installiert][kong:install] werden:

```bash
helm upgrade -i ingress-kong kong \
  --repo https://charts.konghq.com \
  --set ingressController.installCRDs=false
```

Danach kann Kong als Controller einer Ingress Ressource spezifiziert werden:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-kong-ingress
spec:
  ingressClassName: kong
  # ...
```

### Features

Folgende [Features des KIC][kong:features] sind erwähnenswert:

- Routing anhand von Pfaden mit [RegEx][kong:regex]
- Resilienz ([Health-Checking & Circuit-Breaker][kong:resilience])
- Plugins
  - können aus dem [Plugin Hub][kong:plugins:hub] bezogen
    - Authentifizierung (JWT, LDAP, BasicAuth, ...)
    - Autorisierung (ACL)
    - Security (CORS, IP-Blacklist, ...)
    - Traffic Control (Response-Cache, Rate Limiting, ...)
    - Monitoring (Prometheus, ...)
  - oder [selbst erstellt][kong:plugins:custom] werden

### Nachteile

Leider kommt auch der Ingress Controller von Kong nicht gänzlich ohne Nachteile. Zum einen ist die Dokumentation für Plugins etc. nicht für den KIC ausgelegt. So wird in der Dokumentation des [JWT Plugins][kong:plugins:jwt] bspw. lediglich demonstriert, wie das Plugin mithilfe der Kong Admin API aufgesetzt werden kann (wie diese API in K8s exposed wird, muss man selber recherchieren).
Obwohl die wichtigsten Plugins frei zugänglich sind, gibt es auch solche, für die ein [kostenpflichtiges Abonnement][kong:pricing] notwendig ist, bspw. diverse Advanced Plugins oder das [Route By Header Plugin][kong:plugins:route-by-header]

## Skipper Ingress Controller

[Skipper][skipper:github] ist ein HTTP Router, der von Zalando als Open-Source-Software bereitgestellt wird und [als Ingress Controller][skipper:kubernetes] eingesetzt werden kann.

### Installation

Der Skipper Ingress Controller kann entweder als [Daemonset][skipper:install:daemonset] oder als [Deployment][skipper:install:deployment] installiert werden. Dazu muss das Repository geklont und anschließend die jeweiligen Ressourcen via `kubectl create -f ...` erstellt werden.

Alternativ existiert ein [Helm Chart][skipper:helm], welches jedoch zuletzt im Februar 2019 aktualisiert wurde.

### Features

Skipper glänzt mit den [ausführlichen Möglichkeiten][skipper:predicates], wie Traffic geroutet werden kann:

- Host (RegEx)
- Pfad (RegEx)
- Methode (RegEx)
- Header (RegEx)
- Cookies (RegEx)
- Query Parameter (RegEx)
- und noch mehr!

### Nachteile

Trotz meiner besten Bemühungen, konnte ich den Skipper Ingress Controller bei mir lokal (mit Docker Desktop) leider nicht zum laufen bringen. Dazu habe jede Installationsmethode versucht, allerdings wird in dem Helm Chart eine nicht mehr verfügbare API-Version verwendet, weshalb die Installation abgebrochen wird. Das Anwenden der Manifeste aus dem Git Repo funktioniert*, jedoch gelang es mir anschließend nicht, über die in meiner Ingress Ressource definierten Routen zu verwenden. An dieser Stelle musste ich mit Bedauern feststellen, dass im Internet neben der Doku von Zalando so gut wie keine Ressourcen (Blog-Artikel o. ä.) zu Skipper als Ingress Controller zu finden sind.

> *Die in den Manifesten deklarierten Ressourcen werden erstellt. Allerdings werden auch hier deprecated API-Versionen verwendet, welche ab v1.22 nicht mehr verfügbar sein werden. Zur Zeit scheint jedoch ein PR in Arbeit zu sein, welcher die aktuellen API-Versionen inkorporiert. Leider scheint dies wiederum erst für das nächste Major Release geplant zu sein.

## Fazit

In diesem Artikel habe ich meine persönlichen Erfahrungen mit drei Controllern für Kubernetes Ingress dargelegt.
Dabei stellte sich heraus, dass der **NGINX Ingress Controller** von Kubernetes sehr gut für den Einstieg in K8s Ingress geeignet ist. Gründe dafür sind die einfach Installation via Helm sowie die simple Konfiguration, welche ausschließlich über Annotationen in der Ingress Ressource möglich ist.
Der **Kong Ingress Controller** kann ebenso einfach installiert werden, bietet dank Plugins und Custom Resources allerdings noch mehr Features und Konfigurationsmöglichkeiten.
Zalando's **Skipper Ingress Controller** konnte mich leider nicht überzeugen; die Installation ist relativ umständlich, Manifeste sind veraltet und die einzige Informationsquelle ist die Dokumentation von Zalando. Das ist besonders enttäuschend, da die umfangreichen Routing-Methoden sehr vielversprechend waren.

Weitere sehr vielversprechende und beliebte Ingress Controller sind [Istio Ingress][istio] sowie [Traefik][traefik], welche ich zukünftig definitiv auch noch ausprobieren werde.

Ich hoffe, dass dieser Exkurs in Kubernetes Ingress Controllern dem Einen oder Anderen bei der Wahl eines geeigneten Controllers weiterhelfen kann. Im nächsten und letzten Artikel dieser Serie wird erläutert, welche Pattern existieren, um Frontends in einer Microservice-Umgebung zu entwickeln.

<!-- REFERENCES -->
[ingress]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[controller:overview]: https://docs.google.com/spreadsheets/d/191WWNpjJ2za6-nbG4ZoUMXMpUK8KlCIosvQB0f-oq3k
[istio]: https://istio.io/latest/docs/
[traefik]: https://traefik.io/solutions/kubernetes-ingress/
<!-- NGINX -->
[nginx:github]: https://github.com/kubernetes/ingress-nginx
[nginx:official-support]: https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
[nginx:install]: https://kubernetes.github.io/ingress-nginx/deploy/#quick-start
[nginx:annotations]: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/
[nginx:canary]: https://stackoverflow.com/a/70084578/9889501
<!-- Kong -->
[kong:github]: https://github.com/Kong/kubernetes-ingress-controller
[kong:kong]: https://github.com/Kong/kong
[kong:install]: https://docs.konghq.com/kubernetes-ingress-controller/2.0.x/deployment/k4k8s/#helm
[kong:regex]: https://discuss.konghq.com/t/kong-ingress-controller-and-path-based-routing/2956/6
[kong:resilience]: https://docs.konghq.com/kubernetes-ingress-controller/2.0.x/guides/configuring-health-checks/
[kong:features]: https://github.com/Kong/kubernetes-ingress-controller/#features
[kong:pricing]: https://konghq.com/pricing/
[kong:plugins:hub]: https://docs.konghq.com/hub/
[kong:plugins:custom]: https://docs.konghq.com/kubernetes-ingress-controller/2.0.x/guides/setting-up-custom-plugins/
[kong:plugins:jwt]: https://docs.konghq.com/hub/kong-inc/jwt/
[kong:plugins:route-by-header]: https://docs.konghq.com/hub/kong-inc/route-by-header/
<!-- Skipper -->
[skipper:github]: https://github.com/zalando/skipper/
[skipper:kubernetes]: https://opensource.zalando.com/skipper/kubernetes/ingress-controller/
[skipper:install:deployment]: https://opensource.zalando.com/skipper/kubernetes/ingress-controller/#deployment
[skipper:install:daemonset]: https://opensource.zalando.com/skipper/kubernetes/ingress-controller/#daemonset
[skipper:helm]: https://github.com/baez90/skipper-helm
[skipper:predicates]: https://opensource.zalando.com/skipper/reference/predicates/
