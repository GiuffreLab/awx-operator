# Basic AWX Install

You first need to build kustomization.yaml manifest that will build the 
AWX-Operator Controller. 

``` yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=2.7.0
  - awx-pvc.yaml  # Remove this if you are not using a PVC
  - awx-manifest.yaml

# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.7.0

# Specify a custom namespace in which to install AWX
namespace: awx
```

# Basic Environment

You can set up a very basic version of AWX-Operator with this simple yaml file. What you put as the `metadata` name will be the name of the AWX-Operator deployment. This is important if you are deploying more than one AWX-Operator instance to the same namespace. **If you do not want persistent storage, this would be your `awx-manifest.yaml` file.**

create an `awx-manifest.yaml` file with the below contents

``` yaml
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
  namespace: awx
spec:
  service_type: nodeport
  ingress_type: none
  hostname: awx-demo.example.com
```

# Environment with Persistence

When you’re adding persistent storage to your AWX deployment in a  cluster, the steps often depend on the underlying infrastructure and the storage solutions available or already in place.

For a K3s cluster, local path storage is available by default. If you don’t have a more complex storage solution (like NFS, Ceph, etc.), K3s automatically provisions storage from the local node's filesystem. 

Here is the basic setup with the local path provisioner in a K3s cluster using local storage:


**Define a Persistent Volume Claim (PVC)**

create an `awx-pvc.yaml` file withe the below contents

``` yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: awx-pvc
  namespace: awx
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi  # Modify the storage size as per your needs
  storageClassName: standard  # Modify the storage class as per your needs
```

**Define the AWX Manifest**

**This is the `awx-manifest.yaml` file in the repo.**

create an `awx-manifest.yaml` file with the below contents

``` yaml
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
  namespace: awx
spec:
  service_type: nodeport
  ingress_type: none
  hostname: awx-demo.example.com
  tower_postgres_storage_class: standard  # Specify the storage class used by PostgreSQL
  tower_postgres_storage_size: 20Gi  # It should be enough to specify just the storage size
  web_nodeport: 32000  # Pre-define the NodePort for web GUI access (default range is 30000-32767)

```

# Deploy the Kustomization and Manifests

``` bash
kubectl apply -k .
```

you can watch the progress with

``` bash
kubectl get pods -n awx --watch
```

This will take some time to build, and the web GUI will not be available for a few minutes even after the pods show they are Running.

Your final deployment can be verified by running the following, and the results will look similar.

``` bash
kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator" -n awx
NAME                             READY   STATUS    RESTARTS   AGE
awx-demo-postgres-13-0           1/1     Running   0          47h
awx-demo-task-68d5bb94fc-46872   4/4     Running   0          47h
awx-demo-web-594cc6c687-bpfqb    3/3     Running   0          47h
```

# Last steps before logging in

Before you can login you need to confirm your `web_nodeport` for the web GUI (pre-defined in the `awx-manifest.yaml` file)

confirm it by running the following command

```bash
kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
awx-demo-postgres   ClusterIP   None           <none>        5432/TCP       4m4s
awx-demo-service    NodePort    10.109.40.38   <none>        80:32000/TCP   3m56s
```

Note that this deployment shows our `web_nodeport` on the line with `awx-demo-service` as its type is `nodeport` and the internal port is `80` and the external port for GUI access is `32000`


# First time login

Now that you know your Web GUI port, it should be reachable from the host's `IP:Port`

The initial login uses the following credentials

Username: `admin`

The default password is found by running this command and copying the `echo` statement.

``` bash
kubectl get secret awx-demo-admin-password -n awx -o jsonpath="{.data.password}" | base64 --decode ; echo
```
