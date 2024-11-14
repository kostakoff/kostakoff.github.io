# Instruction
## How to prepare microk8s in ubuntu24.10 for application development

### Required resources
- [X] ubuntu 24.10
- [X] 24Gb memory
- [X] 10 cores

### Steps
- prepare ubuntu 24.10 (vm or install to your PC)
- install microk8s, kubectl and helm
```bash
for package in "microk8s --classic" "helm --classic" "kubectl --classic" ; do sudo snap install $package; done
```
- fix permissons to microk8s for non root user
```bash
sudo usermod -a -G microk8s $USER
mkdir ~/.kube
```
- enable important k8s operators for development
```bash
microk8s enable dns
microk8s enable dashboard
microk8s enable storage
microk8s enable ingress
microk8s enable registry
microk8s enable cert-manager
microk8s enable prometheus
```
- check k8s single node cluster status
```bash
microk8s status
```
- save root user kubeconfig
```bash
microk8s config > ~/.kube/config
chmod 600 ~/.kube/config
```
- check that kubectl works
```bash
kubectl version
```
- get kubectl web dashboard token
```bash
kubectl get secret microk8s-dashboard-token --namespace=kube-system -o jsonpath="{.data.token}" | base64 --decode ; echo
```
#### Create ingress for operators
- create dasboard-ingress.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dashboard
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - dashboard.k8s.localhost
  rules:
    - host: dashboard.k8s.localhost
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kubernetes-dashboard
                port:
                  number: 443
```
- create grafana-ingress.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
spec:
  tls:
    - hosts:
        - 'grafana.k8s.localhost'
  ingressClassName: nginx
  rules:
    - host: 'grafana.k8s.localhost'
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kube-prom-stack-grafana
                port:
                  number: 80
```
- create prometeus-ingress.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - prometheus.k8s.localhost
  rules:
    - host: prometheus.k8s.localhost
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: prometheus-operated
                port:
                  number: 9090
```
- deploy ingresses
```bash
kubectl apply -f dasboard-ingress.yaml --namespace=kube-system
kubectl apply -f grafana-ingress.yaml --namespace=observability
kubectl apply -f prometeus-ingress.yaml --namespace=observability
```
- in web browser try to login to kubernetes dashboard https://dashboard.k8s.localhost with token

#### Create self signed certificates issuer
- create selfsigned-cluster-issuer.yaml
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-cluster-issuer
spec:
  selfSigned: {}
```
- deploy selfsigned-cluster-issuer
```bash
kubectl apply -f selfsigned-cluster-issuer.yaml --namespace=cert-manager
```
#### Create custom application namespace
- create applicatoins-ns.yaml
```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: applications
```
- deploy applicatoins namespace
```bash
kubectl apply -f applicatoins-ns.yaml
```
# Documentation
## How to use microk8s single node cluster
- switch to applications namespace
```bash
kubectl config set-context --current --namespace=applications
```
- get current namespace
```bash
kubectl config view --minify -o jsonpath='{..namespace}'; echo
```
Feel free to use it!

### Cluster hot links:
- https://dashboard.k8s.localhost
- https://grafana.k8s.localhost/?orgId=1 (admin/prom-operator)
- https://prometheus.k8s.localhost/targets?search=

### Persistent volumes for microk8s 
- default persistent volumes stogare path: /var/snap/microk8s/common/default-storage
