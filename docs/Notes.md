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
kind: Deployment
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

!!! info "Information"

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

### Services

Services allows you to expose your application you created as pods and deployments so other users can access them.

They are the connectors in your application network - they ensure everything that needs to communicate can do so i.e., internal pods or external resources.

Example:

If you have a groups of pods for an application split into Frontend, Database, and Backend.

These need to commmunicate to each other for the application to run smoothly.

Services allows for seamless connectivity between each of these group of pods.

They allow you to do this without worrying about networking.

They allow applications to be built in a loosely coupled way - so different parts of your application can work independently, and if upgrades are needed, services ensure they can still communicate with each other. This allows modularity in differnt parts of your applications.

Services use labels to identify which pods they should be connecting - so when you create a service you are telling Kubernetes which labels to look out for to connect these group of pods together.

Different types:
- Cluster IP: Default - gives stable internal IP address that pods can use to talk to each other
- Node Port: Exposes your service on a specific port on each node on your cluster, a simple way to make your service exposable outside the cluster
- Load Balancers: Typically used in Cloud environments - needs cluster in cloud. Creates External Load Balancer that distributes traffic to your pods.

!!! info "Information"

    Services work at the Network layer, using TCP and other Networking protocols. These are the backbone of Networking in Kubernetes. Behind the scenes kube-proxy is in charge - creating ip tables and routes.

#### Cluster IP

It is like the internal phone-line of the Kubernetes cluster.

When you create a Service of the type: ClusterIP - Kubernetes gives it a unique internal IP address called the Cluster IP. If pods need to talk to each other but don't need external access.

Example:

If you have front-end pods, but need to communicate with back-end pods and they need to talk to each other, and not the outside world, Cluster IP is suitable.

The IP acts as a middle-man for these pods to talk to each other.

It is secure since ONLY accessible within the cluster, keeping internal communication safe, and is the default service type - so you just create a service and Kubernetes takes care of the rest.

DEMO:

Create deployment + ClusterIP Service

You can do this by using the same code in the deployment.yaml file and adding `---` to create a seperate service

```yaml title="deployment + service yaml file (nginx-svc.yaml)" linenums="1"
apiVersion: apps/v1
kind: Deployment
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
---

apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

`ApiVersion`: Set to v1 for services

`kind`: tells Kubernetes the `Service` object is being created.

`metadata`: Contains informtion such as labels, name, namespaces etc.

`spec`: Defines specification at service level


`selector`: Links Service to the correct pods - matches labels to ones in the `template` labels in the `metadata` - this ensures pods it's routing traffic to are the correct ones.

Service here is connected to the pods using the selector label `app: nginx` which matches the templated pod label `app: nginx`

`ports` contains information about ports related to the Service:

`protocol` is the network protocol being used - set as `TCP`, which is default

`port` is the Service port - running on HTTP (port 80)

`targetPort` is the port for the pod/deployment being targeted (port 80 as defined in aerlier `containerPort`)

`kubectl apply -f nginx-svc.yaml` to create deployment + service

To access the service via ClusterIP you need the Cluster IP:

```bash
$ kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   5h46m
nginx-svc    ClusterIP   10.96.183.89   <none>        80/TCP    21m
```
One way to test the service is to access it through the IP address locally.

For this you need to do a port-forward:

```bash
kubectl port-forward svc/nginx-svc 8080:80
```
Format:
`svc/[NAME OF SERVICE]
[localhost port]:[Service port]

You access the application via localhost/[localhost port]

Another way is to launch a temporary shell with Network debugging tools i.e, curl, wget, dig etc.

You can then test using a relevant tool such as using curl to see if it is accessible

#### Node Port

It is a way to make your service accessible outside the Kubernetes cluster. It opens a specific port on each node in your cluster and forwards traffic from that port to your service. 

That means you can access your service using the node's IP address and the specific node port assigned.

Example:

If you have a nginx web server - it will have a port it typically listens on i.e., 80 - this is the Target Port

You have a service port - this is an internal port the service listens on. Kubernetes forwards requests from this port to the target port on the pods.

Node port - this is the port that gets opened on each node, typically a high number i.e, 30080.

If you want to reach your service from outside the cluster, you would use the node ip and the port - http://nodeip:nodeport

Node port is useful when you want to export the service to the outside world but don't need a load balancer - this makes it useful for development and local testing.


#### Load Balancer

This exposes your service externally using your cloud providers load balancer - this will distribute traffic across your pods. There is an internal load balancer too that distributes traffic evenly to your pods.

## Storage

Persistent storage is required since containers are ephemeral i.e., temporary or short-lived.

For example, if you have a database associated with your pod, and it goes down, then the database goes with it - and new pods won't have that data on it.

Persistent storage ensures data reliability beyond the pod lifecycle, and is critical for stateful applications i.e., applications that need to retain data from previous sessions such as databases, or message queues.

### PV

To address this, Kubernetes uses Persistent Volumes (PVs) which is storage set aside for applications. 

The storage is abstracted - so applications does not need to be concerned with WHERE the storage comes from (cloud, NFS server, local disk etc.). It just needs to know that storage is AVAILABLE, so can use it.

Kubernetes admins can define volumes with properties such as size, access mode, and storage claims.

Once created, they are available for pods to claim through Persistent Volume Claims (PVCs)

PVCs are requests for storage that applications make to the cluster - which include the amount of storage, and other requirements.

The cluster then looks for a Persistent Volume that matches the request and binds it to the PVC.

This allows the application to access storage available in the cluster.

## Configuration Data

### Config Map

If your application needs configuration data to run - it needs to get this from somewhere, such as environment variables, urls, or file paths.

You might not want to hardcode these in the application in cases they change.

So you can store them using ConfigMaps - these are storage boxes for non-confidential data that your pod might need to access, and config settings to your pod.

They provide and pass configuration settings - ensuring they have the correct configuration before they spin up.

DEMO: Config Map

A config map will be created that passed the environment variables `APPCOLOUR=blue` and `APPMODE=production`:

To create use:

`kubectl create configmap my-config --from-literal=APPCOLOUR=blue --from-literal=APPMODE=production`

To pass the config it is passed as part of a pod being created, based on a busybox container.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-demo
spec:
  containers:
  - name: demo-container
    image: busybox
    command: ["/bin/sh", "-c", "env && sleep 3600"]
    env:
    - name: APP_COLOUR
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: APP_COLOUR
    - name: APP_MODE
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: APP_MODE
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: my-config
```

Here a pod is defined which is based on the busybox container. 

Environment variables are passed in using `valueFROM`, which maps the pod environment variables to the ones created in the `my-config` Config Map created earlier by the `key`. 

These values are accessible by defining a volume containing the Config Map in the pod and mounting this onto the container.

### Secrets

These are designed to store secrets - they are base64 encoded.

This means it isn't stored as plain text.

These aren't secure - as someone can simply decode it.

LOOK AT ALTERNATIVE WAYS OF STORING ENCRYPTED SECRETS.

These can be mounted onto containers as volumes or exposed as environment variables.

DEMO: Secret Creation and use

use `kubectl create secret generic my-secret --from-literal=username=myuser --from-literal=password=mypassword`

To check these are applied use:

`kubectl get secrets`

```bash
NAME        TYPE     DATA   AGE
my-secret   Opaque   2      67m
```

To see what this looks like in Kubernetes:

```bash
$ kubectl get secrets my-secret -o yaml
apiVersion: v1
data:
  password: bXlwYXNzd29yZA==
  username: bXl1c2Vy
kind: Secret
metadata:
  creationTimestamp: "2026-05-10T19:14:11Z"
  name: my-secret
  namespace: default
  resourceVersion: "37790"
  uid: 5d59cd85-b9c8-4492-9b33-8522c541a2cb
type: Opaque
```

These are base64 encoded - so the actual values can be found by using base64 decoding:

```bash
$ echo "bXlwYXNzd29yZA==" | base64 -d
mypassword
$ echo "bXl1c2Vy" | base64 -d
myuser
```

Not so secure.

A pod is defined with secrets accessible BOTH as environment variables and volume mounts:

```bash title="secret-pod.yaml" linenums=1
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo-pod
spec:
  containers:
    - name: secret-demo-container
      image: nginx
      env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: username
        - name: SECRET_PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: password
      volumeMounts:
      - name: secret-volume
        mountPath: "etc/secret-volume"
        readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: my-secret
```

To apply:

`kubectl apply -f secret-pod.yaml`

To investigate what's going on exec into the container:

`kubectl exec -it secret-demo-pod -- /bin/sh`

Once inside - you can get the username and passwords through the two methods:


```sh title = "Environment variable:"
# echo $SECRET_USERNAME
myuser
# echo $SECRET_PASSWORD
mypassword
```

```sh title = "Environment variable:"
$ cat /etc/secret-volume/username
myuser
$ cat /etc/secret-volume/password
mypassword
```
