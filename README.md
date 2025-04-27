
# 🛡️ Secure Microservices Architecture with Service Mesh & Network Policies

---

## ✅ Project Objective

Set up a **secure Kubernetes cluster** with:
- Deploy a **sample microservices app** (e.g., Online Boutique)
- Enable **Istio** service mesh with **mutual TLS (mTLS)**
- Apply **Kubernetes Network Policies** to restrict traffic
- Enforce **Pod Security Policies (PSP)** or **OPA Gatekeeper** for container hardening


---

## 🛠️ Tech Stack

- **Kubernetes** (Minikube, Kind, or any cloud provider)
- **Istio** (Service Mesh)
- **Network Policies**
- **Pod Security Policies (PSP)** / **OPA Gatekeeper**
- **Online Boutique** (Google's Microservices Demo App)

---

## 📂 Project Structure

```bash
secure-microservices-architecture/
├── manifests/
│   ├── online-boutique/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   ├── network-policies/
│   │   ├── allow-frontend-to-backend.yaml
│   │   └── default-deny-all.yaml
│   └── psp/
│       └── restricted-psp.yaml
├── istio/
│   ├── istio-install.sh
│   ├── enable-mtls.yaml
│   └── ingress-gateway.yaml
├── README.md
└── setup.sh
```

---

## file-setup.py

```python
import os

# Base directory
base_dir = "/home/lilia/VIDEOS/secure-microservices-architecture"

# Files and their content
files = {
    "manifests/online-boutique/deployment.yaml": "# Placeholder for Online Boutique deployment\napiVersion: apps/v1\nkind: Deployment\nmetadata:\n  name: sample-deployment\nspec:\n  selector:\n    matchLabels:\n      app: sample\n  replicas: 1\n  template:\n    metadata:\n      labels:\n        app: sample\n    spec:\n      containers:\n      - name: sample\n        image: sample-image\n",

    "manifests/online-boutique/service.yaml": "# Placeholder for Online Boutique service\napiVersion: v1\nkind: Service\nmetadata:\n  name: sample-service\nspec:\n  selector:\n    app: sample\n  ports:\n    - protocol: TCP\n      port: 80\n      targetPort: 8080\n",

    "manifests/network-policies/allow-frontend-to-backend.yaml": "apiVersion: networking.k8s.io/v1\nkind: NetworkPolicy\nmetadata:\n  name: allow-frontend-to-backend\nspec:\n  podSelector:\n    matchLabels:\n      app: backend\n  ingress:\n  - from:\n    - podSelector:\n        matchLabels:\n          app: frontend\n",

    "manifests/network-policies/default-deny-all.yaml": "apiVersion: networking.k8s.io/v1\nkind: NetworkPolicy\nmetadata:\n  name: default-deny-all\nspec:\n  podSelector: {}\n  policyTypes:\n  - Ingress\n  - Egress\n",

    "manifests/psp/restricted-psp.yaml": "apiVersion: policy/v1beta1\nkind: PodSecurityPolicy\nmetadata:\n  name: restricted\nspec:\n  privileged: false\n  runAsUser:\n    rule: 'MustRunAsNonRoot'\n  seLinux:\n    rule: 'RunAsAny'\n  supplementalGroups:\n    rule: 'MustRunAs'\n    ranges:\n    - min: 1\n      max: 65535\n  volumes:\n  - 'configMap'\n  - 'emptyDir'\n  - 'projected'\n  - 'secret'\n",

    "istio/istio-install.sh": "#!/bin/bash\n# Install Istio demo profile\nset -e\ncurl -L https://istio.io/downloadIstio | sh -\ncd istio-*\nexport PATH=\"$PWD/bin:$PATH\"\nistioctl install --set profile=demo -y\n",

    "istio/enable-mtls.yaml": "apiVersion: security.istio.io/v1beta1\nkind: PeerAuthentication\nmetadata:\n  name: default\n  namespace: default\nspec:\n  mtls:\n    mode: STRICT\n",

    "istio/ingress-gateway.yaml": "apiVersion: networking.istio.io/v1alpha3\nkind: Gateway\nmetadata:\n  name: online-boutique-gateway\nspec:\n  selector:\n    istio: ingressgateway\n  servers:\n  - port:\n      number: 80\n      name: http\n      protocol: HTTP\n    hosts:\n    - \"*\"\n",

    "README.md": "# \ud83d\udd35 Secure Microservices Architecture with Service Mesh & Network Policies\n\n(Refer to full GitHub README guide above.)\n",

    "setup.sh": "#!/bin/bash\n# Setup script to deploy everything\n\necho \"[*] Starting Secure Microservices Setup...\"\n\nkubectl apply -f manifests/online-boutique/\nkubectl apply -f manifests/network-policies/\nkubectl apply -f manifests/psp/\nkubectl apply -f istio/enable-mtls.yaml\nkubectl apply -f istio/ingress-gateway.yaml\n\necho \"[*] Setup Complete!\"\n"
}

# Create the files
for path, content in files.items():
    full_path = os.path.join(base_dir, path)
    os.makedirs(os.path.dirname(full_path), exist_ok=True)
    with open(full_path, "w") as f:
        f.write(content)

print(f"All files created successfully under {base_dir}!")

```

---

## 📋 Prerequisites

- Kubernetes Cluster (Minikube, Kind, EKS, AKS, or GKE)
- `kubectl` installed
- `istioctl` installed
- (Optional) `helm` installed (for OPA Gatekeeper)

---

## 🚀 Step-by-Step Setup

### Step 1: Create Kubernetes Cluster

```bash
minikube start --kubernetes-version=v1.27.0
```

---

### Step 2: Install Istio Service Mesh

Install Istio:

```bash
curl -L https://istio.io/downloadIstio | sh -
cd istio-*/
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y
```

Enable automatic sidecar injection:

```bash
kubectl label namespace default istio-injection=enabled
```

---

### Step 3: Deploy Sample Microservices App

Clone and deploy Online Boutique app:

```bash
git clone https://github.com/GoogleCloudPlatform/microservices-demo.git
kubectl apply -f microservices-demo/release/kubernetes-manifests.yaml
```

---

### Step 4: Enable mTLS in Istio

Create `istio/enable-mtls.yaml`:

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT
```

Apply it:

```bash
kubectl apply -f istio/enable-mtls.yaml
```

✅ All service-to-service communication is now **encrypted**!

---

### Step 5: Expose Services Securely via Istio Gateway

Create `istio/ingress-gateway.yaml`:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: online-boutique-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```

Apply:

```bash
kubectl apply -f istio/ingress-gateway.yaml
```

---

### Step 6: Apply Network Policies

#### 6.1 Default Deny All Traffic

Create `manifests/network-policies/default-deny-all.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

Apply:

```bash
kubectl apply -f manifests/network-policies/default-deny-all.yaml
```

---

#### 6.2 Allow Frontend-to-Backend Communication

Create `manifests/network-policies/allow-frontend-to-backend.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

Apply:

```bash
kubectl apply -f manifests/network-policies/allow-frontend-to-backend.yaml
```

✅ Now, only specific traffic is allowed between services.

---

### Step 7: Enforce Pod Security Policies (PSP) or OPA Gatekeeper

#### 7.1 Create a Restricted PSP

Create `manifests/psp/restricted-psp.yaml`:

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  supplementalGroups:
    rule: 'MustRunAs'
    ranges:
    - min: 1
      max: 65535
  volumes:
  - 'configMap'
  - 'emptyDir'
  - 'projected'
  - 'secret'
```

Apply:

```bash
kubectl apply -f manifests/psp/restricted-psp.yaml
```

---

#### 7.2 (Alternative) Install OPA Gatekeeper

OPA Gatekeeper is a modern replacement for PSP (recommended for Kubernetes 1.25+):

```bash
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/deploy/gatekeeper.yaml
```

---

## 🔥 Final Outcome

- ✅ Microservices run inside Kubernetes.
- ✅ Service-to-service traffic encrypted using **Istio mTLS**.
- ✅ Strict **Network Policies** isolate pods.
- ✅ **Pod security** ensures no privileged containers run.

---

## 🧹 Cleanup

```bash
kubectl delete -f microservices-demo/release/kubernetes-manifests.yaml
kubectl delete -f istio/enable-mtls.yaml
kubectl delete -f istio/ingress-gateway.yaml
kubectl delete -f manifests/network-policies/
kubectl delete -f manifests/psp/
```

---

## 🎯 Useful Commands

| Purpose | Command |
|:--------|:--------|
| Access Istio Kiali Dashboard | `istioctl dashboard kiali` |
| Check mTLS is enabled | `istioctl authn tls-check` |
| View Network Policies | `kubectl get networkpolicy` |
| Debug Services | `kubectl get svc,pods -A` |

---

## 📈 Future Enhancements

- Add Prometheus + Grafana for monitoring.
- Use Istio AuthorizationPolicies for fine-grained RBAC.
- Enforce namespace-specific mTLS.

---

# 🚀 Happy Securing Microservices!

---

