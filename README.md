# awx setup in kubernetes using awx operator

# Pre-requisite

makesure you have kubernetes cluster running with atleast 4 cpus & 6GB ram.

# Deploying awx operator, postgres, awx

**awx operator installation**

```
kubectl apply -f awx-resources.yaml
```

**Note**: the above command creates `awx` namespace and deploys awx operator in `awx` namespace. Ensure that pod is running state prior proceeding.

**changing the context**

Lets change the context to `awx` namespace by executing the below command.

```
kubectl config set-context --current --namespace=awx
```
**note**: you may need sudo for this command

**Deploying awx instance**

```
kubectl apply -f awx-demo.yaml
```

The above command deploys `postgres` & `awx` instance. This will take some time. So please wait.
Makesure that all the pods are in running state.

# Accessing awx webui

**Forward Ports to access the pod**

there is service named `awx-demo-service` deployed that will need to ports forwarded. Execute `kubectl port-forward` against this service.

from the `awx` namespace run the following

```
kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
```

it will return the following example with a different port

```
NAME                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
awx-demo-postgres   ClusterIP   None          <none>        5432/TCP       34h
awx-demo-service    NodePort    10.43.43.87   <none>        80:31527/TCP   34h
```
currently it is running on port `31527` based on the above results (this is random every time its installed). to change it to a specific port of your choice run the following

```
kubectl edit svc awx-demo-service
```

this will bring up the list of services, look for the following line and change the `nodePort:31527`

```
ports:
  - name: http
    nodePort: 31527
    port: 80
    protocol: TCP
    targetPort: 8052
```

use the `down arrow` then hit the `instert` key, then make the change, hit `esc`, then type `:wq!`

you should now be able to access awx via your web browser. before doing that you need to get the base admin password by entering the following

`kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode`

go to the browser & open the endpoint

username: admin
