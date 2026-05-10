CLUSTER CREATION DEMO:

Kind used to create Kubernetes cluster for development/testing - perfect for learning

Cluster defined - this creates API server needed for pods to be created:


```yaml title="Pod YAML fie" linenums="1"
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker

# Current supported version is v1alpha4 https://kind.sigs.k8s.io/docs/user/configuration/
```



Pods are the smallest unit of deployment in a Kubernetes cluster

Each pod contains one or more containers:

Containers in a pod are always scheduled together, i.e, they share the same machine

They share the same IP address and port space, and communicate to each other using localhost

They can have access to a shared local storage on the node that is hosting the pod.

Containers don't get default access to storage, storage must be explictly mounted to each container inside a pod.

Below is how to create a pod declaratively using the `.yaml` file

DEMO CREATE POD:

Pod `nginx-pod.yaml` file:

```yaml title="Pod YAML fie" linenums="1"
apiVersion: v1
Kind: Pod
Metadata:
    labels:
        run: nginx
    name: nginx
Spec:
    containers:
    -image: nginx
     name: nginx
```

`apiVersion` - specifies the API version, typically v1

`Kind` - specifies the Object, set to `Pod`

`Metadata` - contains identifying information like `name` and optional `labels`

`Spec`- Outlines further information for Kubernetes - since it is a `Pod` object, the `containers` field is defined which speicifies the container to run. It is a list (given by `-` symbol in `-image`), since Pods may have more than one container.

`image` specifies the image to be used - if no tag, then default `latest` tag

To run this file `kubectl apply -f pod.yaml` - this will create the pod

You can also create pod imperatively using the command line - but NOT RECOMMENDED since it is not declarative unlike other method:

`kubectl run nginx --image-nginx`

Can also find out the underlying yaml file using:

`kubectl get pod nginx --output yaml`

Will output:

```yaml title="Pod YAML fie" linenums="1"
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2026-05-10T12:27:35Z"
  generation: 1
  labels:
    run: nginx
  name: nginx
  namespace: default
  resourceVersion: "2024"
  uid: efe0e522-5442-4ed7-a4f2-0cfc637238df
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
.
.
.
```

Deploying apps on Kubernetes is important

For large scale operations, where you need many instances - instead of deploying pods one-by-one you can use a deployment.

This is a powerful Controller which helps you manage your applucations across many instacnes effortlessly. 

It doesn't just create and update pods, it ensures the right number of pods are always running as specified.

i.e., if a pod is deleted or failed, Kubernetes will automatically create antoher one to meet the deployment speicfication.

These are important since 
- They make it easy to manage large-scale operaitions, 
- Scaling: If you need to scale based on traffic, you can do so based on the load, without manual intervention
- Upgrade/rollbacks: This is much easier, if something goes wrong when updating, you can quickly roll back to the previous version. Also support rolling updates.


DEMO deployment.yaml file (`nginx-deployment.yaml`):

```yaml title="Pod YAML fie" linenums="1"
apiVersion: apps/v1
metadata:
    name: nginx-deployment
    labels:
        app: nginx
spec:
    replicas: 3
    selector:
        matchLabels:
            app: nginx
    template:
        metadata:
            labels:
                app: nginx
        spec:
            containers:
            - name: nginx
              image: nginx
            ports:
            - containerPort: 80
```

`ApiVersion`: Set to apps/v1 for managing Deployments and other higher-level Kubernetes objects.

`kind`: tells Kubernetes the `Deployment` object is being created.

`metadata`: Contains informtion such as labels, name, namespaces etc.

`spec`: Defines specification at deployment level

`replicas`: Defines how many copies i.e., replicas of pods you want to run.

Set to 3 here - Kubernetes will ensure that 3 pods are running.

`selector`: Links deployments to the correct pods - matches labels to ones in the `template` labels defined later to ensure pods it's managing are the correct ones.

Deployment here is connected to the `nginx` app using teh label `app: nginx`

`template`: This is the blueprint for the pods to be deployed.

In this case it is the same as the pod yaml file `nginx-pod.yaml` defined earlier.

`metadata`: Contains informtion such as labels, name, namespaces etc.

Here the `labels:` show `app: nginx` - which the selector from earlier used to ensure the correct pods are being used.

`Spec`- Outlines further information for Kubernetes - since it is a `Pod` object, the `containers` field is defined which specifies the container to run. It is a list (given by `-` symbol in `-image`), since Pods may have more than one container.

`image` specifies the image to be used - if no tag, then default `latest` tag

`ports` list of container ports application is running on

To run this deployment `kubectl apply -f nginx-deployment.yaml`

??? info "Information"

    When running this Kubernetes does not just create pods directly - it creates Replica sets FIRST. This is a controller responsible for ensuring the correct number of pods are running. This can be checked by running `kubectl get replicaset` - it shows you the replica set with the same name as your deployment followed by unique identifier. This takes care of creating your pods.

### ReplicaSets

Kubernetes provides different options for controlling the amount of pods running, the simplest controller is a ReplicaSet.

This is a controller that ensures a specified number of pod instances are running at any time.

If one instance of a pod fails, the ReplicaSet automatically spins up a new instance, maintaining the desired state of your applications - even if individual pods fail.

Deployments are a step up from ReplicaSets, and are a superset of these.

These make it even easier by managing ReplicaSets

Differences:

- Rolling updates: These gradually replace your old pods with new ones, minimising downtime i.e., if something goes wrong

You won't interact with ReplicaSets by themselves.

In a deployment you can have 2 versions of application running which you can define, this will then have a ReplicaSet for each version (total 2). These will then control how many pods of each version will be running at a given time.

A deployment simplifies things, by taking care of ReplicaSets - they give you more control with less effort.

