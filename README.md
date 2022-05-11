## Application Design and Build

### Define, build and modify container images
#### Command
```
docker image build .
docker-compose -f dockerfile.yml  up -d --no-recreate
docker exec -it container-name bash
```
### Jobs and CronJob
#### Command

```
kubectl get jobs
kubectl get cronjob
```
#### Job Definition Yaml
- Can create job / cron job in Parallelism
```
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completion: 3
  parallelism: 3
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ['expr','3','+','2']
      restartPolicy: Never
```

### Multi-container Pod Design Pattern
#### Reason for multi-container
- Different type of services that are separated but are created together and destroyed together

#### Ambassador
- When there are multiple service, that pod outsource such logic to a separate container within the POD, so that the application can always refer to a database at localhost, and the new container, will proxy that request to the right database.

#### Adapter
- The adapter container processes the logs with different format of different containers, before sending it to the central server in a centralized format.

#### Sidecar
- Deploying a logging agent along side a web server to collect logs and forward them to a central log server.

#### Sample Yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
      - containerPort: 8080
  
  - name: log-agent
    image: log-agent
```
### Utilize Volume
- 
## Application Deployment
- The deployment provides us with capabilities to upgrade the underlying instances seamlessly using rolling updates, undo changes, and pause and resume changes to deployments.
- Whenever you create a new deployment or upgrade the images in an existing deployment it triggers a Rollout
- A rollout is the process of gradually deploying or upgrading the application containers.
- When upgrading the application, the kubernetes deployment object creates a NEW replicaset under the hoods and starts deploying the containers there. At the same time taking down the PODs in the old replica-set following a RollingUpdate strategy. 

### Common Deployment Strategies
#### Recreate strategy
- Destroy all of old one and then create newer versions of application instances.

#### Rolling Update strategy
- Instead we take down the older version and bring up a newer version one by one.
- Default strategy

### Command
```
kubectl rollout status <deployment name>
kubectl rollout history <deployment name>
kubectl rollout undo <deployment name>
kubectl apply -f <deployment config yaml>
kubectl set image <deployment name>
kubectl get deployments
kubectl describe deployment <deployment name>
```

### Sample Yaml
```
apiVersion: apps/v1
kind: deployment
metadata:
  name: my-deployment
  labels:
    app: myap
    type: frontend
  spec:
    template:
      metadata:
        name: myapp-pod
        labels:
          app: myapp
          type: front-end
        spec:
          containers:
            - name: nginx-container
              image: nginx
        replicas: 3
        selector:
          matchLabels:
            type: front-end
```

### Canary 

### Helm
```
helm install <deployment name>
helm upgrade <deployment name>
helm rollback <deployment name>
helm uninstall <deployment name>
hekm search repo
helm repo add
helm uninstall
```

## Application Observability and Maintenance
- When a POD is first created, it is in a Pending state. This is when the Scheduler tries to figure out were to place the POD. If the scheduler cannot find a node to place the POD, it remains in a Pending state
- Once the POD is scheduled, it goes into a ContainerCreating status, were the images required for the application are pulled and the container starts. Once all the containers in a POD starts, it goes into a running state.
### API Deprecations

### Probes and Health Check
- readinessProbes
  - Kubernetes does not immediately set the ready condition on the container to true, instead, it performs a test to see if the api responds positively.
  - Specify how often to probe by using the periodSeconds option
  -  Make more attempts by using the failureThreshold option
- livenessProbes
  - A liveness probe can be configured on the container to periodically test whether the application within the container is actually healthy. If the test fails, the container is considered unhealthy and is destroyed and recreated.

### Monitor K8 Applications by debugging and utilize logging
- There are a number of open-source solutions available
today, such as the Metrics-Server, Prometheus, the Elastic Stack, and proprietary solutions like Datadog and Dynatrace

```
kubectl logs -f <application name>
kubectl top node
kubectl top pod
kubectl -n elastic-stack exec -it app -- cat /log/app.log
```

### Sample Yaml File
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    name: webapp
spec:
  containers:
    - name: webapp
      image: webapp
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
      livenessProbe:
        httpGet:
          path: /api/ready
          port: 8080
```
## Application Environment, Configuration and Security
### CRD
- 
### Authentication, Authorization and Admission Control
- 
### Define resources requirements, limit and quotas
- 
### ConfigMap
- ConfigMaps are used to pass configuration data in the form of key value pairs in Kubernetes.
- There are two phases involved in configuring ConfigMaps. First create the ConfigMaps and second Inject them into the POD.

#### Imperative
- Without using a ConfigMap definition file
- Example command
```
kubectl create configmap \ 
  app-config --from-literal=APP_COLOR=blue \ 
             --from-literal=APP_MOD=prod
```

#### Declarative
- Using a ConfigMap Definition file
- Example Yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

#### Example in Pod Definition
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: webapp
    image: webapp
    envFrom:
      - configMapRef:
        name: app-config
```

### Secrets
- There are two steps involved in working with Secrets. First, create the secret and second inject it into Pod.

#### Imperative
- Example command
```
kubectl create secret generic \ 
  app-config --from-literal=DB_HOST=blue \ 
             --from-literal=DB_USER=prod
```

#### Declarative
- Convert the data from plain text to an encoded format
```
echo -n 'mysql' | base64
echo -n 'bX1zcWw=' | base64 --decode
```
- Example Yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_HOST: bX1zcWw=
  DB_USER: cm9vdA
```

#### Example in Pod Definition
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: webapp
    image: webapp
    envFrom:
      - secretRef:
        name: app-secret
```

### Service Account
- In order for application to query the kubernetes API, it has to be authenticated. For that a service account is needed.
- Command
```
kubectl create serviceaccount dashboard-sa
kubectl exec -it my-kubernetes-dashboard cat /var/run/secrets/kubernetes.io/serviceaccount/token
kubectl exec -it my-kubernetes-dashboard ls /var/run/secrets/kubernetes.io/serviceaccount
```
- Example Yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
  automountServiceAccountToken: false
```
resources:
  requests:
    memory: "1Gi"
    cpu: 1
### Security Context
- Unlike virtual machines containers are not completely isolated
from their host. Containers and the hosts share the same kernel.
- Containers are isolated using namespaces in Linux. The host has a namespace and the containers have their own namespace. 
- All the processes run by the containers are in fact run on the host itself, but in their own namespaces. 

#### Example Yaml File
```
appVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
```

## Service and Networking

### Network Policies
- A Network policy is another object in the kubernetes namespace. Just like PODs, ReplicaSets or Services. Can apply a network policy on selected pods.
- Not all network solutions support network policies. A
few of them that are supported are kube-router, Calico, Romana and Weave-net.

#### Sample Yaml for Network Policy
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: "true"
    ports:
      - protocol: TCP
        port: 3306
```

### Ingress Rules
- Ingress helps your users access your
application using a single Externally accessible URL, that you can configure to route to different services within your cluster based on the URL path, at the same time terminate TLS. 
- Ingress Resource is a set of rules and configurations applied on the ingress controller
- Specify a set of rules to configure Ingress. The solution you deploy is called as an Ingress Controller. And the set of rules you configure is called as Ingress Resources. Ingress resources are created using definition files like the ones we used to create PODs, Deployments and services earlier in this course.
- Incoming traffic from the users is an Ingress Traffic. And the outgoing requests to the app server is Egress traffic.

#### Sample Yaml for Controller (i)
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress 
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      targetPort: 443
      protocol: TCP
      name: https
```

#### Sample Yaml for Controller (2)
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller 
spec:
  replicas:
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
    labels:
      name: nginx-ingress
    spec:
      containers:
        - name : nginx-controller
          image : xxx
      args:
        - /nginx
        - --configmap
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      targetPort: 443
      protocol: TCP
      name: https
```

#### Sample Yaml for Resources
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress 
spec:
rules:
  - host: wear.my-online-store.com
    http: 
      paths:
        - path: /wear
          backend: 
            serviceName: wear-service
            servicePort: 80
```
#### Command
```
kubectl -n dev get svc
kubectl expose pod redis --port=6379 --name redis-service
kubectl taint nodes node01 spray=mortein:NoSchedule
```


kubectl create job throw-dice-job --image=kodekloud/throw-dice --dry-run=client -o yaml > throw-dice-job.yaml

## Reference
- Learn concepts and practice for the Kubernetes Certification with hands-on labs right in your browser - DevOps - CKAD by Mumshad Mannambeth