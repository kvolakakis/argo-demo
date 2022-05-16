## [Argo Project](https://argoproj.github.io/) demo - CS548

### Content
* Argo Workflows
* Argo CD
* Argo Events
* Argo Rollouts (not implemented)

> Before we start:
```
helm repo add argo https://argoproj.github.io/argo-helm

kubectl create namespace argo
```
### Argo Workflows

> Install Argo Workflows and get it working: 
```
helm install argo-workflows argo/argo-workflows -n argo

## get credentials (use Git Bash or linux-based terminal)
SECRET=$(kubectl get sa -n argo argo-workflows-server -o=jsonpath='{.secrets[0].name}')
ARGO_TOKEN="Bearer $(kubectl get secret -n argo $SECRET -o=jsonpath='{.data.token}' | base64 --decode)"
echo $ARGO_TOKEN

kubectl port-forward -n argo deployment/argo-workflows-server 2746:2746

## goto http://127.0.0.1:2746/workflows/ and use credentials to sign-in
```

> Octopus Workflow YAML (examples/workflows/argo-dag.yaml):
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: plokamia-argo-
spec:
  entrypoint: plokamia
  templates:
  - name: echo
    inputs:
      parameters:
      - name: message
    container:
      image: alpine:3.7
      command: [echo, "{{inputs.parameters.message}}"]
  - name: plokamia
    dag:
      tasks:
      - name: FOREHEAD
        template: echo
        arguments:
          parameters: [{name: message, value: FOREHEAD}]
      - name: EYE-RIGHT
        dependencies: [FOREHEAD]
        template: echo
        arguments:
          parameters: [{name: message, value: EYE-RIGHT}]
      - name: NOSE
        dependencies: [EYE-LEFT, EYE-RIGHT]
        template: echo
        arguments:
          parameters: [{name: message, value: NOSE}]
      - name: EYE-LEFT
        dependencies: [FOREHEAD]
        template: echo
        arguments:
          parameters: [{name: message, value: EYE-LEFT}]
      - name: CHEEK-RIGHT
        dependencies: [EYE-RIGHT, NOSE]
        template: echo
        arguments:
          parameters: [{name: message, value: CHEEK-RIGHT}]
      - name: CHEEK-LEFT
        dependencies: [EYE-LEFT, NOSE]
        template: echo
        arguments:
          parameters: [{name: message, value: CHEEK-LEFT}]
      - name: NECK
        dependencies: [CHEEK-LEFT, CHEEK-RIGHT]
        template: echo
        arguments:
          parameters: [{name: message, value: NECK}]
      - name: TENTACLE1
        dependencies: [NECK]
        template: echo
        arguments:
          parameters: [{name: message, value: TENTACLE1}]
      - name: TENTACLE2
        dependencies: [NECK]
        template: echo
        arguments:
          parameters: [{name: message, value: TENTACLE2}]
      - name: TENTACLE3
        dependencies: [NECK]
        template: echo
        arguments:
          parameters: [{name: message, value: TENTACLE3}]
      - name: TENTACLE4
        dependencies: [NECK]
        template: echo
        arguments:
          parameters: [{name: message, value: TENTACLE4}]
      - name: TENTACLE5
        dependencies: [NECK]
        template: echo
        arguments:
          parameters: [{name: message, value: TENTACLE5}]
      - name: TENTACLE6
        dependencies: [NECK]
        template: echo
        arguments:
          parameters: [{name: message, value: TENTACLE6}]
      - name: TENTACLE7
        dependencies: [NECK]
        template: echo
        arguments:
          parameters: [{name: message, value: TENTACLE7}]
      - name: TENTACLE8
        dependencies: [NECK]
        template: echo
        arguments:
          parameters: [{name: message, value: TENTACLE8}]
```
### Argo CD

> Install Argo CD and get it working:
```
helm install argo-cd argo/argo-cd -n argo

## Port forward to have access to the service 
kubectl port-forward -n argo service/argo-cd-argocd-server 8080:443

## get credentials (use Git Bash or linux-based terminal)
kubectl -n argo get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

## login from terminal (if needed)
./argocd login 127.0.0.1:8080 --username admin --password {TOKEN/password}
```

### Argo Events

```
helm install argo-events argo/argo-events -n argo 

cd example/events

kubectl apply -f argo-events-event-bus.yaml -n argo
kubectl apply -f argo-events-event-source.yaml -n argo

curl -X POST \
    -H "Content-Type: application/json" \
    -d '{"message":"My first webhook"}' \
    http://localhost:12000/example

kubectl apply -f argo-events-plokamia-sensor.yaml -n argo

curl -X POST \
    -H "Content-Type: application/json" \
    -d '{"message":"My first webhook"}' \
    http://localhost:12000/example

kubectl get workflows -n argo
## also see workflows UI

## to avoid conflicts with next example (using same event-source)
kubectl delete -f argo-events-plokamia-sensor.yaml -n argo

## does not work yet (even though it should). Check it out @kelado.
kubectl apply -f argo-events-createpod-sensor.yaml -n argo

kubectl get pods -n argo

curl -X POST \
    -H "Content-Type: application/json" \
    -d '{"message":"My first webhook"}' \
    http://localhost:12000/example
    
kubectl logs [created_pod_name] -n argo ## expect to see message sent with curl
```

> Argo CD Deployment (examples/cd/argo-cd-deployment.yaml) and Service (examples/cd/argo-cd-svc.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rollouts-example
spec:
  replicas: 5
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: rollouts-example
  template:
    metadata:
      labels:
        app: rollouts-example
    spec:
      containers:
      - image: kvolakakis/rollouts-example:v2.0
        name: rollouts-example
        ports:
        - containerPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: rollouts-example
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: rollouts-example
```

> Argo CD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: test
spec:
  destination:
    name: ''
    namespace: default
    server: 'https://kubernetes.default.svc'
  source:
    path: examples/rollouts
    repoURL: 'https://github.com/kvolakakis/argo-demo'
    targetRevision: HEAD
  project: default

```


