Pods are the smallest unit of deployment in a Kubernetes cluster

Each pod contains one or more containers:

Containers in a pod are always scheduled together, i.e, they share the same machine

They share the same IP address and port space, and communicate to each other using localhost

They can have access to a shared local storage on the node that is hosting the pod.

Containers don't get default access to storage, storage must be explictly mounted to each container inside a pod.


Pod .yaml file:

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

To run this file `kubectl apply -f pod.yaml`
