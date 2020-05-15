# Authors : Jeremy Zerbib, Samuel Mettler
# Date : 21st of May 2020
# Labo 06 : Kubernetes

## Task 1 - Deploy the application on a local test cluster

### DEPLOY THE REDIS SERVICE AND POD

Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report.

Quand on a essayé de déployer Redis en utilisant les fichiers `.yaml` nous avons eu l'erreur suivante : 
```bash
error: error validating "https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/rbac/heapster-rbac.yaml": error validating data: unknown object type schema.GroupVersionKind{Group:"rbac.authorization.k8s.io", Version:"v1beta1", Kind:"ClusterRoleBinding"}; if you choose to ignore these errors, turn validation off with --validate=false
```
Après avoir cherché une solution (aussi indiqué dans le message d'erreur), on a decidé d'ajouter le paramètre `--validate=false`. La commande est donc :
```bash
kubectl create -f redis-svc.yaml --validate=false
```
Même chose pour la commande d'après avec `redis-pod.yaml`

Nous avons également eu du mal lors du déploiement du `api-pod.yaml` car nous étions pas certain des informations que l'on devait mettre dedans. En effet nous savions pas si l'attribut `component` devait avoir comme value `api` ou `redis`. Après avoir essayé avec `api` nous avons vu une erreur lors du déploiement : `Containers with unready status` nous avons changé pour `redis` et cela a marché.

	Copy the object descriptions into the lab report : `api-pod.yaml`
```
apiVersion: v1
kind: Pod
metadata:
  name: api
  labels:
    component: api
    app: todo
spec:
  containers:
  - name: api
    image: icclabcna/ccp2-k8s-todo-api
    ports:
    - containerPort: 8081
    resources:
      limits:
        cpu: 300m
    env:
    - name: REDIS_ENDPOINT
      value: redis-svc
    - name: REDIS_PWD
      value: ccp2

```
### DEPLOY THE FRONTEND POD 

Pour le `frontend-pod.yaml` la valeur de la variable d'environnement `API_ENDPOINT_URL` doit être l'IP du svc qui est retrouvable dans le dashboard, c'est à dire : 10.0.0.108

	Copy the object descriptions into the lab report : `frontend-pod.yaml`
```
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    component: api
    app: todo
spec:
  containers:
  - name: api
    image: icclabcna/ccp2-k8s-todo-frontend
    ports:
    - containerPort: 8080
    resources:
      limits:
        cpu: 300m
    env:
    - name: API_ENDPOINT_URL
      value: 10.0.0.108
```

## Task 2 - Deploy the application in kubernetes engine

## Task 3 - Add and exercise resilience

