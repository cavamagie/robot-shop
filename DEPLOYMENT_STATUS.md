# Deployment Status - Robot Shop Services

## Stato Attuale (2026-06-15)

### ✅ Servizi Creati e Pronti

Tutti i seguenti servizi sono stati creati con successo nel repository:

1. **Inventory Service** - `inventory/`
2. **Recommendation Service** - `recommendation/`
3. **Legacy Java Service** - `legacy-java/`
4. **Secure Web Server** - `secure-web/`

### 🔄 Build in Corso

**Problema Attuale:** I build su OpenShift stanno incontrando limiti di rate di Docker Hub per il pull delle immagini base.

**Build Status:**
```
NAME           TYPE     FROM             STATUS                            
inventory-1    Docker   Binary@55292e2   Failed (PullBuilderImageFailed)
secure-web-1   Docker   Binary@55292e2   Running (rate limit issues)
```

**Errore:** `toomanyrequests: You have reached your unauthenticated pull rate limit`

### 📋 BuildConfig Create su OpenShift

Le seguenti BuildConfig sono state create con successo nel namespace `robot-shop`:

- ✅ `inventory` - BuildConfig creata
- ✅ `recommendation` - BuildConfig creata  
- ✅ `secure-web` - BuildConfig creata
- ✅ `legacy-java` - BuildConfig creata

### 📦 Deployment Manifests Pronti

Tutti i manifest YAML sono pronti per il deployment:

- `K8s/inventory-deployment.yaml`
- `K8s/recommendation-deployment.yaml`
- `K8s/legacy-java-deployment.yaml`
- `K8s/secure-web-deployment.yaml` (include Route con TLS passthrough)

---

## Come Completare il Deployment

### Opzione 1: Attendere e Riprovare

Docker Hub resetta i limiti di rate ogni 6 ore. Attendere e riprovare:

```bash
# Riavvia i build falliti
oc start-build inventory --follow
oc start-build recommendation --follow
oc start-build legacy-java --follow
oc start-build secure-web --follow
```

### Opzione 2: Usare un Registry Alternativo

Modificare i Dockerfile per usare immagini da registry alternativi:

**Per inventory e recommendation (Debian base):**
```dockerfile
# Invece di: FROM debian:10
FROM quay.io/centos/centos:stream9
# oppure
FROM registry.access.redhat.com/ubi8/ubi:latest
```

**Per secure-web (NGINX):**
```dockerfile
# Invece di: FROM nginx:alpine
FROM quay.io/bitnami/nginx:latest
# oppure
FROM registry.access.redhat.com/rhscl/nginx-116-rhel7
```

**Per legacy-java (Debian 9):**
```dockerfile
# Invece di: FROM debian:9
FROM quay.io/centos/centos:7
```

### Opzione 3: Usare Docker Hub con Autenticazione

Creare un secret con le credenziali Docker Hub:

```bash
# Creare secret per Docker Hub
oc create secret docker-registry dockerhub-secret \
  --docker-server=docker.io \
  --docker-username=<your-username> \
  --docker-password=<your-password> \
  --docker-email=<your-email> \
  -n robot-shop

# Linkare il secret al service account builder
oc secrets link builder dockerhub-secret -n robot-shop

# Riavviare i build
oc start-build inventory
oc start-build recommendation
oc start-build legacy-java
oc start-build secure-web
```

### Opzione 4: Build Locale e Push

Buildare le immagini localmente e pushare al registry interno di OpenShift:

```bash
# Ottenere il route del registry interno
REGISTRY=$(oc get route default-route -n openshift-image-registry -o jsonpath='{.spec.host}')

# Login al registry
docker login -u $(oc whoami) -p $(oc whoami -t) $REGISTRY

# Build e push inventory
cd inventory
docker build -t $REGISTRY/robot-shop/inventory:latest .
docker push $REGISTRY/robot-shop/inventory:latest

# Build e push recommendation
cd ../recommendation
docker build -t $REGISTRY/robot-shop/recommendation:latest .
docker push $REGISTRY/robot-shop/recommendation:latest

# Build e push legacy-java
cd ../legacy-java
docker build -t $REGISTRY/robot-shop/legacy-java:latest .
docker push $REGISTRY/robot-shop/legacy-java:latest

# Build e push secure-web
cd ../secure-web
docker build -t $REGISTRY/robot-shop/secure-web:latest .
docker push $REGISTRY/robot-shop/secure-web:latest
```

---

## Deployment dei Servizi

Una volta che le immagini sono state buildare con successo:

```bash
# Verificare che le immagini siano disponibili
oc get imagestream -n robot-shop

# Deployare i servizi
oc apply -f K8s/inventory-deployment.yaml
oc apply -f K8s/recommendation-deployment.yaml
oc apply -f K8s/legacy-java-deployment.yaml
oc apply -f K8s/secure-web-deployment.yaml

# Verificare i deployment
oc get pods -n robot-shop
oc get services -n robot-shop
oc get routes -n robot-shop
```

---

## Verifica dei Servizi

### Inventory Service
```bash
INVENTORY_URL=$(oc get route inventory -o jsonpath='{.spec.host}' 2>/dev/null || echo "not-deployed")
curl http://$INVENTORY_URL/health
curl http://$INVENTORY_URL/inventory
```

### Recommendation Service
```bash
RECOMMENDATION_URL=$(oc get route recommendation -o jsonpath='{.spec.host}' 2>/dev/null || echo "not-deployed")
curl http://$RECOMMENDATION_URL/health
curl http://$RECOMMENDATION_URL/recommendations
```

### Legacy Java Service
```bash
LEGACY_URL=$(oc get route legacy-java -o jsonpath='{.spec.host}' 2>/dev/null || echo "not-deployed")
curl http://$LEGACY_URL/
curl http://$LEGACY_URL/info
```

### Secure Web Server
```bash
SECURE_WEB_URL=$(oc get route secure-web -o jsonpath='{.spec.host}' 2>/dev/null || echo "not-deployed")
curl -k https://$SECURE_WEB_URL/
curl -k https://$SECURE_WEB_URL/health
```

---

## Troubleshooting

### Verificare i log dei build
```bash
oc logs -f bc/inventory
oc logs -f bc/recommendation
oc logs -f bc/legacy-java
oc logs -f bc/secure-web
```

### Verificare i log dei pod
```bash
oc logs -f deployment/inventory
oc logs -f deployment/recommendation
oc logs -f deployment/legacy-java
oc logs -f deployment/secure-web
```

### Cancellare e ricreare i build
```bash
# Cancellare build falliti
oc delete build inventory-1 secure-web-1

# Riavviare i build
oc start-build inventory --follow
oc start-build secure-web --follow
```

---

## Riepilogo

**Servizi Pronti per il Deployment:**
- ✅ Inventory Service (codice completo)
- ✅ Recommendation Service (codice completo)
- ✅ Legacy Java Service (codice completo)
- ✅ Secure Web Server (codice completo)

**BuildConfig Create:**
- ✅ Tutte le BuildConfig sono state create su OpenShift

**Deployment Manifests:**
- ✅ Tutti i manifest YAML sono pronti

**Stato Attuale:**
- ⏳ Build in attesa di completamento (rate limit Docker Hub)
- 📝 Deployment manifests pronti per essere applicati

**Prossimi Passi:**
1. Risolvere il problema dei rate limit (vedere Opzioni 1-4 sopra)
2. Completare i build delle immagini
3. Applicare i deployment manifests
4. Verificare che tutti i servizi siano running
5. Testare gli endpoints

---

## Documentazione

Per maggiori dettagli, consultare:
- [`JAVA_SERVICES.md`](JAVA_SERVICES.md) - Documentazione servizi Java
- [`SECURITY_TESTING.md`](SECURITY_TESTING.md) - Guida security testing
- [`README.md`](README.md) - Documentazione generale