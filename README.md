
# Inroduction
[Kubernetes Mastery](https://www.udemy.com/course/kubernetesmastery)
* Create Shell: `kubectl apply -f https://k8smastery.com/shpod.yaml`
* Attach: `kubectl attach --namespace=shpod -ti shpod`
* Delete at end of course: `kubectl delete -f https://k8smastery.com/shpod.yaml`

```
kubectl get nodes -o json | 
    jq ".items[] | {name:.metadata.name} + .status.capacity"
```

* kubectl describe node/<node>
* kubectl describe node <node>
* `kubectl describe node/docker-desktop`

# Introspection vs Documentation
Exploring Resource Types
* kubectl api-resources
* kubectl explain <type>
kubectl explain node
kubectl explain node.spec
kubectl explain node --recursive

* kubectl get
a stable endpoint to connect to "something"
kubectl get services
kubectl get pods

## Namespaces basics
* kubectl get namespaces

`NAME              STATUS   AGE`
`default           Active   47h`
`kube-node-lease   Active   47h`
`kube-public       Active   47h`
`kube-system       Active   47h`

* get pods --all-namespaces
* get pods -A

can use `-n <namespace>` on many commands

# Section 6: Your First Deployment
Starting in version 1.18 (released March 2020), the kubectl run **command only does one thing: create single pods**. There were many reasons for this, but the big ones were to reduce the complexity of how the run command worked and to move other resource creation to the kubectl create command. The idea is that kubectl run is now similar to docker run. It creates a single pod, where docker run creates a single container.

A majority of us won't be able to use 1.18 for months if not a year or longer, many clouds and production Kubernetes products don't even support 1.17 (or 1.16) yet. I feel it's still important to know the old way that run has always worked, for at least the rest of 2020.

I've decided not to replace all the videos just yet, and to keep the "old" way of doing things intact, while giving you info on the differences in these text lectures.

Depending on which version of Kubernetes you have installed, you'll need to decide how you'll create objects. Here's a cheat sheet for how old commands should be used with the 1.18 changes.

kubectl run nginx --image nginx created a Deployment named nginx before 1.18 (which creates a ReplicaSet, which creates a Pod)

kubectl run nginx --image nginx creates a Pod named nginx in 1.18+

Creating a Deployment in 1.18: kubectl create deployment nginx --image nginx

WARNING: One limit in 1.18 is there is no way to kubectl create deployment and also run a custom command in an image. You can create pods with custom commands using the new kubectl run, but if you want to override the Dockerfile CMD in a deployment, you'll need to use a YAML manifest to do it. This may affect a few examples in this course until all videos are replaced with "1.18 compatible examples".

kubectl run pingpong --image alpine ping 1.1.1.1
kubectl create deployment

* kubectl create cronjob 
restart and run container on a regular basis.

`kubectl create cronjob --schedule="*/3 * * * *" --restart=OnFailure --image=alpine sleep 10`

## kubectl create <resource>
* `kubectl create deployment`
* `kubectl create job`
* `kubectl create cronjob`

## kubectl apply -f <foo.yml>
declarative, giving a state, will try to match state

## multiple nodes
* `kubectl scale deployment pingpong --replicas=8`
* `kubectl logs -l run=pingpong --tail 1 -f`

## Better CLI Logs with Stern
* `brew install stern`
* `stern --tail 1 --timestamps pingpong`

# Section 8: K8s Services and Visualizing Deployments
![Deployment Visualisation]("images/Deployment Visual.png")