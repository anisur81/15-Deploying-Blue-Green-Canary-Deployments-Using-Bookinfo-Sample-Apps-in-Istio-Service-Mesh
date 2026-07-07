# Istio Installation Guide (Step-by-Step)

## 1. Overview

Istio is a service mesh that provides traffic management, security, and observability for microservices running on Kubernetes. This guide explains how to install Istio using the demo profile for learning and testing.

---

## 2. Download Istio

```bash
$ curl -L https://istio.io/downloadIstio | sh -
```

---

## 3. Move to Istio Directory

Example:
```bash
$ cd istio-1.30.0
```

---

## 4. Add istioctl to PATH

**Linux/macOS:**
```bash
export PATH=$PWD/bin:$PATH
```

---

## 5. Install Istio (Demo Profile)

Install Istio using demo profile:
```bash
istioctl manifest apply --set profile=demo
```

**Alternative installation with cert-manager integration:**
```bash
istioctl install -y --set profile=demo --set values.global.caAddress="cert-manager-istio-csr.cert-manager.svc:443"
```

---

## 6. What is Installed in Demo Profile

### 6.1 Control Plane

**Istiod** is the central control component of Istio. It manages:
- Service discovery
- Configuration
- Certificate management (mTLS)
- Sidecar injection rules

### 6.2 Data Plane

**Envoy sidecar proxy** is injected into each workload pod. It handles:
- Traffic routing
- Load balancing
- mTLS encryption
- Telemetry collection

### 6.3 Gateways

- **Istio Ingress Gateway**: Handles incoming external traffic
- **Istio Egress Gateway**: Manages outbound traffic leaving the mesh (optional but included in demo)

### 6.4 Observability Stack

The demo profile includes monitoring and observability tools:
- **Prometheus**: Metrics collection
- **Grafana**: Visualization dashboards
- **Jaeger**: Distributed tracing
- **Kiali**: Service mesh topology and visualization

---

## 7. Installation Output

When running the installation command, you should see output similar to:

```bash
istioctl manifest apply --set profile=demo
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateway installed
✔ Egress gateway installed
✔ Addons installed: Prometheus, Grafana, Kiali, Jaeger
```

---

## 8. Enable Sidecar Injection

**Enable namespace injection:**
```bash
kubectl label namespace default istio-injection=enabled
```

**Verify:**
```bash
kubectl get namespace -L istio-injection
```

---

## 9. Verify Istio Installation

Check the status of Istio components in the `istio-system` namespace:

```bash
k8s@mstr1:~/istio-1.30.0$ kubectl get pods -n istio-system
NAME                                    READY   STATUS    RESTARTS   AGE
istio-egressgateway-64d76c9487-kfrq6    1/1     Running   0          10m
istio-ingressgateway-7c4fcf8489-qtmzg   1/1     Running   0          10m
istiod-6ffcf45f8b-srdxr                 1/1     Running   0          11m
```

---

## 10. Enable Observability Tools

Access the observability dashboards using the following commands:

### Kiali Dashboard
```bash
istioctl dashboard kiali
```

### Grafana Dashboard
```bash
istioctl dashboard grafana
```

### Jaeger Dashboard
```bash
istioctl dashboard jaeger
```

### Prometheus Dashboard
```bash
istioctl dashboard prometheus
```

---

## 11. Install Kubernetes Gateway API CRDs

The Kubernetes Gateway API CRDs do not come installed by default on most Kubernetes clusters, so make sure they are installed before using the Gateway API.

### Install Gateway API CRDs

If they are not already present:
```bash
kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
{ kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.5.1" | kubectl apply -f -; }
```

### Verify Gateway API CRDs Installation

```bash
k8s@mstr1:~/istio-1.30.0$ kubectl get crd gateways.gateway.networking.k8s.io
NAME                                 CREATED AT
gateways.gateway.networking.k8s.io   2026-05-24T15:57:57Z
```

---

## 12. Next Steps

After successful installation, you can:

1. **Deploy sample applications** to test the service mesh
2. **Configure traffic routing** rules using VirtualServices and DestinationRules
3. **Enable mTLS** for secure service-to-service communication
4. **Set up monitoring** and alerting using the observability stack
5. **Implement security policies** using AuthorizationPolicies

---

## 13. Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| **istioctl not found** | Ensure `istioctl` is in your PATH or use the full path to the binary |
| **Pods stuck in Init** | Check if sidecar injection is enabled on the namespace |
| **Gateway not working** | Verify Gateway API CRDs are installed |
| **Observability tools not accessible** | Check if the services are running in `istio-system` namespace |

### Useful Debugging Commands

```bash
# Check Istio configuration
istioctl analyze

# Check sidecar injection status
kubectl get namespace -L istio-injection

# View Istio components logs
kubectl logs -n istio-system deployment/istiod

# Check gateway status
kubectl get gateway -A
```

---

## 📌 Summary

- ✅ Istio installed using **demo profile**
- ✅ Control plane (**Istiod**) running
- ✅ Data plane (**Envoy**) configured for sidecar injection
- ✅ Ingress and Egress gateways deployed
- ✅ Observability stack (**Prometheus, Grafana, Jaeger, Kiali**) installed
- ✅ Gateway API CRDs installed for advanced ingress management
- ✅ Sidecar injection enabled on default namespace

---
 
