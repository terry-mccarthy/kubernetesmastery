
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
![Deployment Visualisation2](images\deployment-visual.png)


# Section 10: Networking Model
## Assignment: Deployments with Services
1. Create a deployment called littletomcat using the tomcat image.
    * `kubectl create deployment littletomcat --image tomcat`

1. What command will help you get the IP address of that Tomcat server?
   * `kubectl get po -l app=littletomcat -o go-template='{{range .items}}{{.status.podIP}}{{"\n"}}{{end}}'`

1. What steps would you take to ping it from another container? (Use the shpod environment if necessary)

   * Create Shell: `kubectl apply -f https://k8smastery.com/shpod.yaml`
   * Attach: `kubectl attach --namespace=shpod -ti shpod`
   * ping \<ip>

1. What command would delete the running pod inside that deployment?
   * `kubectl delete pod littletomcat-7b6c88c67d-5rmdt --now`

1. What happens if we delete the pod that holds Tomcat, while the ping is running?

   * `ping will fail`
   * when new pod comes up it will have a new IP

1. What command can give our Tomcat server a stable DNS name and IP address? <br>
(An address that doesn't change when something bad happens to the container)
   ```
   kubectl expose deployment littletomcat --port 8080
   ```

1. What commands would you run to curl Tomcat with that DNS address?<br>
(Use the shpod environment if necessary)

   * Attach: kubectl attach --namespace=shpod -ti shpod
   * curl http://littletomcat.default:8080

1. If we delete the pod that holds Tomcat, does the IP address still work? How could we test that?

   * if you use the ClusterIP from within the cluster (shpod)
   * `ClusterIP=$(kubectl get services/littletomcat -o go-template='{{(.spec.clusterIP)}}')`
   * ClusterIP was not pingable?? `10.100.x.x`. Refer [StackOverflow](https://stackoverflow.com/questions/35742070/cannot-ping-clusterip-from-inside-the-pod-and-dns-is-not-working-for-external-do)
   * `ip route get <ClusterIP>` (returns 10.1.x.x on the same range as the pods).
   * this IP is pingable, and when killing the pod it continues to be reachable


# Section 12: Walking through app deployments
[httping ](https://github.com/BretFisher/httping-docker_)

1. What deployment commands did you use to create the pods?
   * `kubectl create deploy web --image bretfisher/wordsmith-web`
   * `kubectl create deploy words --image bretfisher/wordsmith-words`
   * `kubectl create deploy db --image bretfisher/wordsmith-db`

1. What service commands did you use to make the pods available on a friendly DNS name?
   * `kubectl expose deploy/words --port=8080`
   * `kubectl expose deploy/db --port=5432`
   * `kubectl expose deploy/web --type=NodePort --port=80`

1. If we add more wordsmith-words API pods, then when the browser is refreshed, you'll see different words. What is the command to scale that deployment up to 5 pods? Test it to ensure a browser refresh shows different words.

   * `kubectl scale deploy/words --replicas=5`


# Section 13: Shifting from CLI to YAML
`kubectl apply -f https://k8smastery.com/dockercoins.yaml`

## Running an (insecure) dasboard
`kubectl apply -f https://k8smastery.com/insecure-dashboard.yaml`
* needed to update ui image `image: kubernetesui/dashboard:v2.5.1` to be compatible with kubenetes version: v1.22.5
```
Warning: spec.template.spec.nodeSelector[beta.kubernetes.io/os]: deprecated since v1.14; use "kubernetes.io/os" instead
deployment.apps/kubernetes-dashboard created
Warning: spec.template.metadata.annotations[seccomp.security.alpha.kubernetes.io/pod]: deprecated since v1.19; use the "seccompProfile" field instead
```
`kubectl get svc dashboard`

# Section `4: Daemon sets and Lebel basics
* daemons guaruntee runs on every node (eg for monitoring)
* can be controlled more by lables and selectors 

## Creating yaml for our daemon set
* `kubectl get deploy/rng -o yaml > rng.yaml`
* `kubectl apply -f rng.yaml`
```
error: error validating "rng.yaml": error validating data: [ValidationError(DaemonSet.spec): unknown field "progressDeadlineSeconds" in io.k8s.api.apps.v1.DaemonSetSpec, ValidationError(DaemonSet.spec): unknown field "replicas" in io.k8s.api.apps.v1.DaemonSetSpec, ValidationError(DaemonSet.spec): unknown field "strategy" in io.k8s.api.apps.v1.DaemonSetSpec, ValidationError(DaemonSet.status): unknown field "availableReplicas" in io.k8s.api.apps.v1.DaemonSetStatus, ValidationError(DaemonSet.status.conditions[0]): unknown field "lastUpdateTime" in io.k8s.api.apps.v1.DaemonSetCondition, ValidationError(DaemonSet.status.conditions[1]): unknown field "lastUpdateTime" in io.k8s.api.apps.v1.DaemonSetCondition, ValidationError(DaemonSet.status): unknown field "readyReplicas" in io.k8s.api.apps.v1.DaemonSetStatus, ValidationError(DaemonSet.status): unknown field "replicas" in io.k8s.api.apps.v1.DaemonSetStatus, ValidationError(DaemonSet.status): unknown field "updatedReplicas" in io.k8s.api.apps.v1.DaemonSetStatus, ValidationError(DaemonSet.status): missing required field "currentNumberScheduled" in io.k8s.api.apps.v1.DaemonSetStatus, ValidationError(DaemonSet.status): missing required field "numberMisscheduled" in io.k8s.api.apps.v1.DaemonSetStatus, ValidationError(DaemonSet.status): missing required field "desiredNumberScheduled" in io.k8s.api.apps.v1.DaemonSetStatus, ValidationError(DaemonSet.status): missing required field "numberReady" in io.k8s.api.apps.v1.DaemonSetStatus]; if you choose to ignore these errors, turn validation off with --validate=false
```
* `kubectl apply -f rng.yaml --validate=false`
* this works and creates additional pod (same name is ok for different types)

## Selectors
Are the glue for connecting resources to each other. How it finds the pods it will send connections to (pool of pods to round robin connections to)
`kubectl describe svc rng`

`kubectl get pods -l app=rng`
`kubectl get pods --selector app=rng`

