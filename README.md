# 🔬 Vulnerable K8s Lab — CVE Detection Testing

> **⚠️ USO ESCLUSIVO IN AMBIENTI DI TEST ISOLATI**
> Non deployare in produzione o su cluster esposti a Internet.

---

## CVE inclusi

| File | Componente | CVE | CVSS | Tipo |
|------|-----------|-----|------|------|
| `01-log4shell.yaml` | Apache Solr 8.11.0 | CVE-2021-44228 | **10.0** CRITICAL | RCE via JNDI lookup in log4j 2.14.1 |
| `02-struts-cve.yaml` | Apache Struts 2.3.31 | CVE-2017-5638 | **10.0** CRITICAL | RCE via Content-Type header multipart |
| `03-openssl-cve.yaml` | nginx 1.20.2 / OpenSSL 1.1.1l | CVE-2022-0778 | 7.5 HIGH | Infinite loop in BN_mod_sqrt (DoS) |
| `03-openssl-cve.yaml` | nginx 1.20.2 / OpenSSL 1.1.1l | CVE-2022-1292 | 9.8 CRITICAL | Shell injection in c_rehash |
| `03-openssl-cve.yaml` | nginx 1.20.2 / OpenSSL 1.1.1l | CVE-2021-3449 | 7.4 HIGH | NULL ptr deref via renegotiation |
| `04-python-vuln-deps.yaml` | Python / requests, urllib3, certifi | CVE-2023-32681 | 6.1 MEDIUM | Header leak su redirect HTTP |
| `04-python-vuln-deps.yaml` | Python / urllib3 1.26.4 | CVE-2021-33503 | 7.5 HIGH | ReDoS in urllib3 |
| `04-python-vuln-deps.yaml` | Python / certifi < 2022.12.7 | CVE-2022-23491 | 6.5 MEDIUM | CA bundle compromessa |
| `05-node-vuln-deps.yaml` | Node.js / protobufjs 6.11.2 | CVE-2022-25878 | 8.8 HIGH | Prototype pollution |
| `05-node-vuln-deps.yaml` | Node.js / axios 0.21.1 | CVE-2021-3749 | 7.5 HIGH | ReDoS |
| `05-node-vuln-deps.yaml` | Node.js / follow-redirects 1.14.7 | CVE-2022-0536 | 6.5 MEDIUM | Info disclosure |

---

## Deploy

### Tutto in una volta (kustomize)
```bash
kubectl apply -k .
```

### Singoli file
```bash
kubectl apply -f 00-namespace.yaml
kubectl apply -f 01-log4shell.yaml
kubectl apply -f 02-struts-cve.yaml
kubectl apply -f 03-openssl-cve.yaml
kubectl apply -f 04-python-vuln-deps.yaml
kubectl apply -f 05-node-vuln-deps.yaml
```

### Verifica stato pod
```bash
kubectl get pods -n vuln-test
kubectl get svc  -n vuln-test
```

---

## Teardown (rimozione completa)
```bash
kubectl delete namespace vuln-test
```

---

## Suggerimenti per scanner CVE

### Trivy (image scanning)
```bash
# Scansiona tutte le immagini dei pod nel namespace
kubectl get pods -n vuln-test -o jsonpath='{range .items[*]}{.spec.containers[*].image}{"\n"}{end}' \
  | sort -u \
  | xargs -I{} trivy image {}
```

### Grype (Anchore)
```bash
grype solr:8.11.0
grype nginx:1.20.2
grype python:3.9-slim
```

### Snyk
```bash
snyk container test solr:8.11.0
snyk container test nginx:1.20.2
```

### Falco (runtime detection)
Per Log4Shell e Struts assicurati che le regole JNDI e spawn-shell siano attive.

---

## Note tecniche

- Il pod **python-vuln-deps** installa i pacchetti a runtime via `pip install` nell'`args`
  così lo scanner di immagini non li vede nell'immagine base — usa **runtime SCA** o scansiona
  il ConfigMap con le dipendenze.
- Il pod **node-vuln-deps** usa un initContainer per `npm install` in un emptyDir; anche qui
  lo scanner deve agire a runtime o sul SBOM generato.
- Tutti i pod sono privi di `securityContext` → utili anche per test di policy Kubernetes (OPA/Kyverno).
