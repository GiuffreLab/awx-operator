# Deploy AWX-Operator on a Kubernetes node

# Release Version to install

This set of instructions should install the latest release of AWX. As of `04.16.2022` it will install the following

AWX-Operator for kubernetes release version `0.20.0`

AWX release version `20.1.0`

It has been tested on the following server distrobutions
- `Ubuntu`
- `Rocky Linux`
- `Debian`
- `Fedora`

# Pre-requisite

makesure you have kubernetes running on a host with atleast 4 cpus & 6GB ram.

# Deploying awx operator, postgres, awx

**AWX-Operator installation**

```
kubectl apply -f awx-resources.yaml
```

**Note**: the above command creates an `awx` namespace and deploys awx operator. ensure that the pod is in a running state prior to proceeding. you can watch in real time with

**Changing the context**

Lets change the context to `awx` namespace by executing the below command.

```
kubectl config set-context --current --namespace=awx
```

you are now working in the `awx` namespace

**note**: you may need `sudo` for this command if it fails

**Watch the Pods being built**

```
watch kubectl get pods
```

hit `ctrl-c` to exit when you see the two pods running


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

currently it is running on port `31527` based on the above results (this is altered on `line 934` in the `awx-resources.yaml` file)


you should now be able to access `AWX-Operator` via web browser on your chosen port such as `http://<host-ip>:31527`

It will take a few minutes for the database to build before the web page access works some time and it should eventually start working.

**Getting password and logging in**

if you need to get the default admin password you can do so by entering the following

```
kubectl get secret awx-demo-admin-password -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'
```

username: admin