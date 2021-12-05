# Deploy AWX-Operator on a Kubernetes node

# Version to install

`0.15.0`

# Pre-requisite

makesure you have kubernetes running on a host with atleast 4 cpus & 6GB ram.

# Deploying awx operator, postgres, awx

**AWX-Operator installation**

```
kubectl apply -f awx-resources.yaml
```

**Note**: the above command creates an `awx` namespace and deploys awx operator. ensure that the pod is in a running state prior to proceeding. you can watch in real time with

```
watch kubectl get pods
```

hit `ctrl-c` to exit



**Changing the context**

Lets change the context to `awx` namespace by executing the below command.

```
kubectl config set-context --current --namespace=awx
```

you are now working in the `awx` namespace

**note**: you may need `sudo` for this command if it fails



**Deploying the awx instance**

```
kubectl apply -f awx-demo.yaml
```

The above command deploys a `postgres` & `awx` instance. you can watch with the get pods command again



# Accessing awx webui


**Forward Ports to access the pod**

there is service named `awx-demo-service` deployed that you will need to have ports forwarded before you can access it

from the `awx` namespace run the following

```
kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
```

it will return the following example showing what `port` `awx-demo-service` is running on

```
NAME                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
awx-demo-postgres   ClusterIP   None          <none>        5432/TCP       34h
awx-demo-service    NodePort    10.43.43.87   <none>        80:31527/TCP   34h
```

currently it is running on port `31527` based on the above results (this is random every time its installed). to change it to a specific port of your choice (recommend `30000-32767` range for rancher / kubernetes) run the following

```
kubectl edit svc awx-demo-service
```

this will bring up the list of configuration variables, look for the below section and change the `nodePort:31527` line to the `port` you wish

```
ports:
  - name: http
    nodePort: 31527
    port: 80
    protocol: TCP
    targetPort: 8052
```

use the `down arrow` then hit the `instert` key, then make the change, hit `esc`, then type `:wq!`

verify your `port` change with

```
kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
```

as an example it is now on `port 32000`

```
NAME                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
awx-demo-postgres   ClusterIP   None          <none>        5432/TCP       34h
awx-demo-service    NodePort    10.43.43.87   <none>        80:32000/TCP   34h
```

next you will need to forward the `port` of `awx-demo-service` by doing the following in a separate terminal that you do not `ctrl-c` out of. just close the terminal once finished

```
kubectl port-forward service/awx-demo-service 80:32000
```


it should return something like this

```
Forwarding from 127.0.0.1:80 -> 32000
Forwarding from [::1]:80 -> 32000
```


**Getting password and logging in**

you should now be able to access `AWX-Operator` via web browser on your chosen port such as `http://<host-ip>:32000`. before doing that you need to get the default admin password by entering the following

`kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode`

username: admin
