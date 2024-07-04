# Basic AWX Install

You first need to build `kustomization.yaml` manifest that will build the 
AWX-Operator Controller. 

``` yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=2.19.1
  - awx-manifest.yaml

  # Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.19.1

  # Specify a custom namespace in which to install AWX
namespace: awx
```

# Basic Environment

You can set up a very basic version of AWX-Operator with this simple yaml file. What you put as the `metadata` name will be the name of the AWX-Operator deployment. This is important if you are deploying more than one AWX-Operator instance to the same namespace.

create an `awx-manifest.yaml` file with the below contents

``` yaml
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport
  nodeport_port: 32000    # Adjust the port to the port number you wish
  ingress_type: none
  hostname: awx-demo.example.com
  postgres_init_container_resource_requirements: {}
  postgres_data_volume_init: true
```

# Deploy the Kustomization and Manifests

Create the namespace

``` bash
kubectl create namespace awx
```

Deploy the resources

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
```

``` bash
NAME                             READY   STATUS    RESTARTS   AGE
awx-demo-postgres-13-0           1/1     Running   0          47h
awx-demo-task-68d5bb94fc-46872   4/4     Running   0          47h
awx-demo-web-594cc6c687-bpfqb    3/3     Running   0          47h
```

# First time login

The initial login uses the following credentials

Username: `admin`

The default password is found by running this command and copying the `echo` statement.

``` bash
kubectl get secret awx-demo-admin-password -n awx -o jsonpath="{.data.password}" | base64 --decode ; echo
```
