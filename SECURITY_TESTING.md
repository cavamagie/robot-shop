# Security Testing Services

This document describes the additional services created for security testing and demonstration purposes in the Robot Shop application.

## Services Overview

### 1. Secure Web Server
**Location:** `secure-web/`

A secure NGINX-based web server configured with self-signed TLS certificates.

**Features:**
- Self-signed TLS certificate generated at build time
- Dual port configuration (HTTP:8080, HTTPS:8443)
- TLS 1.2 and TLS 1.3 support
- Health check endpoints
- OpenShift Route with TLS passthrough

**Endpoints:**
- `http://secure-web:8080/` - HTTP endpoint
- `https://secure-web:8443/` - HTTPS endpoint (self-signed cert)
- `/health` - Health check endpoint

**Certificate Details:**
- Subject: `/C=IT/ST=Italy/L=Rome/O=RobotShop/OU=IT/CN=secure-web.robot-shop.svc.cluster.local`
- Validity: 365 days
- Type: Self-signed

---

### 2. Legacy Java Service
**Location:** `legacy-java/`

⚠️ **WARNING:** This service intentionally contains outdated and vulnerable dependencies for security testing purposes. **DO NOT USE IN PRODUCTION.**

**Purpose:**
- Security scanning demonstrations
- Vulnerability detection testing
- CVE identification practice
- Security tool validation

**Known Vulnerabilities:**

| Component | Version | CVE/Issue |
|-----------|---------|-----------|
| Log4j | 2.14.1 | CVE-2021-44228 (Log4Shell) |
| Jackson | 2.9.8 | Multiple deserialization CVEs |
| Commons Collections | 3.2.1 | Deserialization vulnerability |
| Spring Boot | 1.5.22 | End of Life, multiple CVEs |
| Tomcat | 8.5.31 | Multiple security issues |
| MySQL Connector | 5.1.45 | Outdated, security issues |
| Apache HttpClient | 4.3.6 | Multiple CVEs |
| Spring Security | 4.2.3 | Outdated, security issues |

**Endpoints:**
- `GET /` - Service information and vulnerability list
- `GET /health` - Health check
- `GET /info` - System information

**Technology Stack:**
- Spring Boot 1.5.22 (EOL)
- Java 8
- Maven
- Debian 9 base image

---

## Deployment

### Prerequisites
- OpenShift CLI (`oc`) installed
- Logged in to OpenShift cluster
- `robot-shop` namespace created

### Deploy Secure Web Server

```bash
# Create binary build
oc new-build --binary --name=secure-web --strategy=docker

# Start build from local directory
oc start-build secure-web --from-dir=./secure-web --follow

# Deploy the service
oc apply -f K8s/secure-web-deployment.yaml

# Verify deployment
oc get pods -l app=secure-web
oc get route secure-web
```

### Deploy Legacy Java Service

```bash
# Create binary build
oc new-build --binary --name=legacy-java --strategy=docker

# Start build from local directory
oc start-build legacy-java --from-dir=./legacy-java --follow

# Deploy the service
oc apply -f K8s/legacy-java-deployment.yaml

# Verify deployment
oc get pods -l app=legacy-java
oc get route legacy-java
```

---

## Testing

### Test Secure Web Server

```bash
# Get the route URL
SECURE_WEB_URL=$(oc get route secure-web -o jsonpath='{.spec.host}')

# Test HTTPS endpoint (will show certificate warning due to self-signed cert)
curl -k https://$SECURE_WEB_URL/

# Test health endpoint
curl -k https://$SECURE_WEB_URL/health
```

### Test Legacy Java Service

```bash
# Get the route URL
LEGACY_URL=$(oc get route legacy-java -o jsonpath='{.spec.host}')

# Get service info and vulnerability list
curl http://$LEGACY_URL/

# Get system information
curl http://$LEGACY_URL/info

# Test health endpoint
curl http://$LEGACY_URL/health
```

---

## Security Scanning

### Scan for Vulnerabilities

Use security scanning tools to detect vulnerabilities in the legacy service:

```bash
# Using Trivy
trivy image legacy-java:latest

# Using Snyk
snyk test --docker legacy-java:latest

# Using Clair
clairctl analyze legacy-java:latest

# OpenShift built-in scanning
oc get imagestreamtag legacy-java:latest -o json | jq '.image.dockerImageMetadata.Config.Labels'
```

### Expected Findings

The legacy service should trigger alerts for:
- Critical: Log4Shell (CVE-2021-44228)
- High: Multiple Jackson deserialization vulnerabilities
- High: Commons Collections deserialization issues
- Medium: Outdated Spring Boot version
- Medium: Tomcat security issues
- Various: Outdated dependencies

---

## OpenShift Routes

### Secure Web Route

The secure-web service uses **TLS passthrough** termination:
- Traffic is encrypted end-to-end
- Certificate is managed by the pod
- OpenShift route passes through encrypted traffic
- Insecure traffic is redirected to HTTPS

```yaml
tls:
  termination: passthrough
  insecureEdgeTerminationPolicy: Redirect
```

### Legacy Java Route

The legacy-java service uses standard HTTP:
- No TLS termination
- Direct HTTP traffic
- Suitable for internal testing

---

## Cleanup

To remove the services:

```bash
# Delete deployments
oc delete -f K8s/secure-web-deployment.yaml
oc delete -f K8s/legacy-java-deployment.yaml

# Delete build configs
oc delete bc secure-web legacy-java

# Delete image streams
oc delete is secure-web legacy-java
```

---

## Security Best Practices

### DO:
✅ Use these services only in isolated test environments
✅ Scan regularly for vulnerabilities
✅ Keep security tools updated
✅ Document all known vulnerabilities
✅ Use for training and education purposes

### DON'T:
❌ Deploy legacy service in production
❌ Expose legacy service to the internet
❌ Use self-signed certificates in production
❌ Ignore security warnings
❌ Store sensitive data in these services

---

## Additional Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CVE Database](https://cve.mitre.org/)
- [OpenShift Security Guide](https://docs.openshift.com/container-platform/latest/security/index.html)
- [Spring Boot Security](https://spring.io/guides/gs/securing-web/)
- [TLS Best Practices](https://wiki.mozilla.org/Security/Server_Side_TLS)

---

## License

These services are provided for educational and testing purposes only. Use at your own risk.