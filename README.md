### Deployment Instructions

**Recommended OS:** Debian 12

1. **Deploy a Kubernetes cluster with a networking plugin**  
   (Make sure you have a working container service)

   Suggestion: [K3s Quick Start Guide](https://docs.k3s.io/quick-start)

   ```bash
   curl -sfL https://get.k3s.io | sh -
   ```

2. **Install `istioctl`**  
   [Istioctl Installation Docs](https://istio.io/latest/docs/ops/diagnostic-tools/istioctl/)

   ```bash
   curl -sL https://istio.io/downloadIstioctl | sh -
   export PATH=$HOME/.istioctl/bin:$PATH
   ```

3. **Install Istio with ingress and egress gateways**

   ```bash
   istioctl install \
     --set profile=default \
     --set components.egressGateways[0].name=istio-egressgateway \
     --set components.egressGateways[0].enabled=true
   ```

4. **Delete Traefik LoadBalancer (to use Istio ingress instead)**

   ```bash
   kubectl delete svc traefik -n <traefik-namespace>
   ```

5. **Add Redis DB to cluster with Helm**

   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update
   helm install redis bitnami/redis \
     --set auth.enabled=false \
     --set architecture=standalone
   ```

6. **Generate TLS certificate & key and create Kubernetes secret**

   ```bash
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
     -keyout tls.key -out tls.crt \
     -subj "/CN=<your.k8s.domain>/O=<yourorg>"

   kubectl create -n istio-system secret tls kamino-cert \
     --key=tls.key --cert=tls.crt
   ```

7. **Set all values in `proclone-secrets.yaml` to your correct secret strings**

8. **Replace all indicated values throughout config files**  
   (e.g. hostnames, IP ranges, domains)

9. **Apply all configuration files**

   ```bash
   kubectl apply -f <your-config-directory>/
   ```

### Argo CD Deployment
**Requires [Argo CD](https://argo-cd.readthedocs.io/en/stable/) and [Argo CD Image Updater](https://argocd-image-updater.readthedocs.io/en/stable/)**

1. **Follow steps 1-6 above**

2. **Set all values in `proclone-argocd-template.yaml`**
   (e.g. hostnames, IP ranges, domains)

3. **Set all values in `proclone-secrets.yaml` to your correct secret strings**

4. **Create the application using the `argocd` cli tool**

   ```bash
   argocd app create --file proclone-argocd-template.yaml
   ```

5. **Apply `proclone-secrets.yaml`**

   ```bash
   kubectl apply -f proclone-secrets.yaml
   ```