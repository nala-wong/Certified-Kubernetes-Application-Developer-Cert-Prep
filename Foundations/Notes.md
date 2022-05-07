## Pods
### Notes

### Command
- Find the number of pods
    ```
    kubectl get pods
    ```
- Find the detilas of pods        
    ```  
    kubectl describe pod <pod name>
    ```
## Replica Sets

### Notes

### Command
- Get the number of replica sets
    ```
    kubectl get rs
    ```

- Create replica sets
    ```
    kubectl apply -f <config.yaml>
    ```

## Deployment

### Notes
- For upgrade docker e.g. rolling updates
- Kind will be change to "Deployment" in yaml file

### Command
- Get the number ofdeployments
    ```
    kubectl get deployments
    ```
- Create deployment
    ```
    kubectl create -f 
    ```
## Namespace

### Notes
- Default, kube-system, kube-public
- Create new name space, can assign the resources 
- Append the namespace if want to use other name space service
- use namespace definition file to create a new space 
- network connectivity need to follow up!!

### Command
- Get DNS


kubectl -n dev get svc


## Imperative Command
kubectl expose pod redis --port=6379 --name redis-service