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

```bash
NAME                     READY     STATUS    		RESTARTS   	AGE
api-68484bc867-vqkzb     1/1       Running   		   0         1m
api-68484bc867-wlsvm     1/1       Running   		   0         1m
front-54966d7858-94v9z   1/1       Running   		   0         1m
front-54966d7858-qwpx2   1/1       Running   		   0         1m
redis-795cb4457b-jspqj   1/1       Running   		   0         2m
front-54966d7858-94v9z   1/1       Terminating  	   0         2m
front-54966d7858-2bntz   0/1       Pending   		   0         35s
front-54966d7858-2bntz   0/1       Pending   		   0         35s
front-54966d7858-2bntz   0/1       ContainerCreating   0         35s
front-54966d7858-94v9z   0/1       Terminating   	   0         2m
front-54966d7858-94v9z   0/1       Terminating   	   0         2m
front-54966d7858-94v9z   0/1       Terminating   	   0         2m
front-54966d7858-2bntz   1/1       Running   		   0         35s
redis-795cb4457b-jspqj   1/1       Terminating   	   0         3m
redis-795cb4457b-8ffs9   0/1       Pending   		   0         35s
redis-795cb4457b-8ffs9   0/1       Pending   		   0         35s
redis-795cb4457b-8ffs9   0/1       ContainerCreating   0         36s
redis-795cb4457b-jspqj   0/1       Terminating   	   0         3m
redis-795cb4457b-8ffs9   1/1       Running   		   0         36s
redis-795cb4457b-jspqj   0/1       Terminating   	   0         3m
redis-795cb4457b-jspqj   0/1       Terminating   	   0         3m
```

### Task 3.2

- What happens if you delete a Frontend or API Pod? How long does it take for the system to react?
  - Cela prend environ 35 secondes pour que le système réagisse et crée un nouveau Pod.
- What happens when you delete the Redis Pod?
  - Quand un pod Redis est supprimé, Google en crée un autre, et ensuite supprime le Pod souhaité afin de ne pas laisser le service tourné sans Pods
- How can you change the number of instances temporarily to 3? Hint: look for scaling in the deployment documentation
  - En tapant la commande suivante, par exemple :`kubectl scale deployment.v1.apps/front-deployment --replicas=3`.  
- What autoscaling features are available? Which metrics are used?
  - Les fonctionnalités d'autoscaling sont :
    - `min` : Nombres de Pods min
    - `max` : Nombres de Pods max
    - `cpu-percent` La base de pourcentage des pods existants
- How can you update a component? (see "Updating a Deployment" in the deployment documentation)
  - 