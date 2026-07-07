 
# Istio Gateway API Implementation Guide

## 1. Verify Istio Supports Gateway API

Check available GatewayClasses:

```bash
kubectl get gatewayclass
```

**Expected Output:**
```
NAME           CONTROLLER                    ACCEPTED   AGE
istio          istio.io/gateway-controller   True       2d
istio-remote   istio.io/unmanaged-gateway    True       2d
```

---

## 2. Create Kubernetes Gateway

### Gateway Configuration
**File:** `gateway-api.yaml`

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: bookinfo-gateway
  namespace: gateway-system
spec:
  gatewayClassName: istio
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: All
```

### Apply Gateway
```bash
kubectl apply -f gateway-api.yaml
```

---

## 3. Create HTTPRoute

### HTTPRoute Configuration
**File:** `httproute.yaml`

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: bookinfo-route
  namespace: bookinfo
spec:
  parentRefs:
  - name: bookinfo-gateway
    namespace: gateway-system
  rules:
  - matches:
    - path:
        type: Exact
        value: /productpage
    - path:
        type: PathPrefix
        value: /static
    - path:
        type: Exact
        value: /login
    - path:
        type: Exact
        value: /logout
    - path:
        type: PathPrefix
        value: /api/v1/products
    backendRefs:
    - name: productpage
      namespace: bookinfo
      port: 9080
```

### Apply HTTPRoute
```bash
kubectl apply -f httproute.yaml
```

---

## 4. Verify Gateway External IP

Check gateway status:

```bash
kubectl get gateway -n gateway-system
```

**Expected Output:**
```
NAME               CLASS   ADDRESS       PROGRAMMED   AGE
bookinfo-gateway   istio   172.18.0.60   True         6h9m
```

---

## 5. Access Bookinfo Application

Access the application using the external IP:

```
http://<GATEWAY-IP>/productpage
```

**Example:**
```
http://172.18.0.61/productpage
```

---

## 6. Benefits of Gateway API

| Benefit | Description |
|---------|-------------|
| **Kubernetes Standard** | Vendor-neutral API |
| **Cleaner Routing** | Easier to use CRDs |
| **Better Portability** | Works across multiple controllers |
| **Future Direction** | Istio is moving toward Gateway API |
| **Better Separation** | Clear separation between infra and app teams |

---

## 7. Check Endpoints

Verify endpoint slices:

```bash
kubectl get endpointSlice
```

**Expected Output:**
```
NAME                ADDRESSTYPE   PORTS   ENDPOINTS                                 AGE
details-r7qm2       IPv4          9080    10.244.73.74                              2d
kubernetes          IPv4          6443    172.18.0.25                               13d
nginx-9rmgr         IPv4          80      10.244.223.1,10.244.73.65                 13d
productpage-l8sqq   IPv4          9080    10.244.223.11                             2d
ratings-8lrh9       IPv4          9080    10.244.223.9                              2d
reviews-j478p       IPv4          9080    10.244.73.75,10.244.73.76,10.244.223.10   2d
```

---

## 8. Verify Gateway and HTTPRoute Status

Check gateway and HTTPRoute resources:

```bash
kubectl get gateways.gateway.networking.k8s.io -n gateway-system
kubectl get httproutes -n bookinfo
```

**Expected Output:**
```
NAME               CLASS   ADDRESS       PROGRAMMED   AGE
bookinfo-gateway   istio   172.18.0.60   True         6h28m

NAME             HOSTNAMES   AGE
bookinfo-route               6h27m
```

---

## 9. Traffic Flow Explanation

```
┌─────────────────────────────────────────────────────────────────┐
│                    Traffic Flow Architecture                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. User sends request (browser/curl)                          │
│     ↓                                                          │
│  2. Request reaches MetalLB external IP                        │
│     ↓                                                          │
│  3. Gateway API Gateway receives the request                   │
│     ↓                                                          │
│  4. HTTPRoute evaluates routing rules                          │
│     ↓                                                          │
│  5. Traffic forwards to productpage Kubernetes service         │
│     ↓                                                          │
│  6. Service routes traffic to Bookinfo application pods        │
│     ↓                                                          │
│  7. Response returns to the client                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. Troubleshooting Guide

### Common Issues and Solutions

| Issue | Troubleshooting Steps |
|-------|----------------------|
| **No External IP** | - Ensure MetalLB is installed and configured<br>- Check MetalLB address pool availability<br>- Verify gateway status: `kubectl get gateway -n gateway-system` |
| **Istio Ingress Gateway Not Running** | - Check pod status: `kubectl get pods -n istio-system`<br>- Verify Istio installation<br>- Check pod logs for errors |
| **Gateway/HTTPRoute Not Accepted** | - Validate YAML syntax<br>- Check status conditions: `kubectl describe gateway bookinfo-gateway -n gateway-system`<br>- Verify API versions are correct |
| **Bookinfo Pods Not Healthy** | - Check pod status: `kubectl get pods -n bookinfo`<br>- Verify sidecar injection is enabled<br>- Check pod logs for application errors |
| **Routing Not Working** | - Verify HTTPRoute rules are correct<br>- Check service endpoints are available<br>- Inspect Istio proxy logs<br>- Test with curl using correct paths |
| **Sidecar Injection Not Working** | - Verify namespace label: `kubectl get ns bookinfo -o yaml`<br>- Check for `istio-injection=enabled` label<br>- Ensure Istio sidecar injector is running |

### Additional Debugging Commands

```bash
# Check Istio proxy logs
kubectl logs <pod-name> -n bookinfo -c istio-proxy

# Verify service endpoints
kubectl get endpoints -n bookinfo

# Check gateway status conditions
kubectl describe gateway bookinfo-gateway -n gateway-system

# Verify HTTPRoute status
kubectl describe httproute bookinfo-route -n bookinfo

# Test routing with curl
curl -v http://<GATEWAY-IP>/productpage
```

### Prerequisites Checklist

- [ ] MetalLB installed and configured
- [ ] Istio installed with Gateway API support
- [ ] Bookinfo application deployed
- [ ] Namespace labels set for sidecar injection
- [ ] Gateway resource created
- [ ] HTTPRoute resource created
- [ ] Service and pods are healthy

---

## 11. Quick Reference

| Resource | Command |
|----------|---------|
| Check GatewayClasses | `kubectl get gatewayclass` |
| List Gateways | `kubectl get gateway -A` |
| List HTTPRoutes | `kubectl get httproute -A` |
| Check Endpoints | `kubectl get endpoints -A` |
| View Gateway Status | `kubectl describe gateway <name> -n <namespace>` |
| View HTTPRoute Status | `kubectl describe httproute <name> -n <namespace>` |
| Check Pods | `kubectl get pods -n <namespace>` |

---

## 12. Cleanup

To remove all resources created in this guide:

```bash
# Delete HTTPRoute
kubectl delete httproute bookinfo-route -n bookinfo

# Delete Gateway
kubectl delete gateway bookinfo-gateway -n gateway-system

# Optional: Delete Bookinfo application
kubectl delete ns bookinfo
```
