
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

# Section 15: Editing Resource Selectors
Want to remove a pod from the load balancer.
* The mission of a *replica set* is:
   * Make sure there is the right number of pods matching this spec!
   * selector: `app=rng,pod-template-hash=abcd1234`
* The mission of a *daemon set* is:
   * Make sure there is a pod matching this spec on *each* node.
   * selector: `app=rng,controller-revision-hash=abcd1234`

## Complex seclectors
* multiple labels are understood as a logical AND

## Editing pod labels
1. add label `enabled=yes` to our `rng` pods
1. update selector for `rng` service to also include `enabled=yes`
1. toggle traffic to a pod by adding/removing `enabled` label

* `kubectl label pods -l app=rng enabled=yes`
* `kubectl edit service rng`
* `POD=$(kubectl get pods -l app=rng,pod-template-hash -o name)`
* `kubectl logs --tail 1 --follow $POD`
* remove the label `kubectl label pod -l app=rng,pod-template-hash enabled-`

## Assignment
Our goal here will be to create a service that load balances connections to two different deployments. You might use this as a simplistic way to run two versions of your apps in parallel for a period of time.

In the real world, you'll likely use a 3rd party load balancer to provide advanced blue/green or canary-style deployments, but this assignment will help further your understanding of how service selectors are used to find pods and use them as service endpoints.

For simplicity, version 1 of our application will be using the NGINX image, and version 2 of our application will be using the Apache image. They both listen on port 80 by default and have different HTML by default so that it's easy to distinguish which is being accessed.

Once properly set up, when we connect to the service we expect to see some requests being served by NGINX and some requests being served by Apache.

### Objectives:
We need to create two deployments: one for v1 (NGINX), another for v2 (Apache).

They will be exposed through a single service.

The selector of that service will need to match the pods created by *both* deployments.

For that, we will need to change the deployment specification to add an extra label, to be used solely by the service.

That label should be different from the pre-existing labels of our deployments, otherwise, our deployments will step on each other's toes.

We're not at the point of writing our own YAML from scratch, so you'll need to use the kubectl edit command to modify existing resources.

### Questions for this assignment
What commands did you use to perform the following?

1. Create a deployment running one pod using the official NGINX image.
   * `kubectl create deployment v1 --image=nginx`

1. Expose that deployment.
   * `kubectl expose deploy/v1 --type=NodePort --port=80`

1. Check that you can successfully connect to the exposed service.
   * `PORT=$(kubectl get svc/v1 -o go-template='{{(index .spec.ports 0).nodePort}}')`
   * `curl localhost:$PORT` should return html including `Thank you for using nginx.`

What commands did you use to perform the following?

1. Change (edit) the service definition to use a label/value of myapp: web
   * `kubectl edit svc/v1`
   * modify the selector from `app: v1` to `myapp: "web"`

1. Check that you cannot connect to the exposed service anymore.
   * `curl localhost:$PORT` should return `Empty reply from server`

1. Change (edit) the deployment definition to add that label/value to the pod template.
   * `kubectl edit deploy v1`
   * modify `template.metadata.label` to add `myapp: "web"`

1. Check that you *can* connect to the exposed service again.
   * `curl localhost:$PORT` 

What commands did you use to perform the following?

1. Create a deployment running one pod using the official Apache image (httpd).
   * `kubectl create deployment v2 --image=httpd`

1. Change (edit) the deployment definition to add the label/value picked previously.
   * `kubectl edit deploy/v2`
   * modify `template.metadata.label` to add `myapp: "web"`

1. Connect to the exposed service again.
   * `curl localhost:$PORT` will round robin between the deployments
   * nginx: `Thank you for using nginx.`
   * apache: `It works!`

(It should now yield responses from both Apache and NGINX.)

# Section 16: All about YAML
Each manifest needs four parts:
```
apiVersion: # find with kubectl api-versions
kind:       # find with kubectl api-resources
metadata:
spec:       # find with kubectl describe pod
```

* `-o yaml --dry-run` will give you example yaml
* `kubectl explain <kind>.spec`

* yamllint.com
* codebeautify.org/yaml-validator
* kubeyaml.com
* pip install yamllint
* kubeval - `github.com/instrumenta/kubeval`

# Section 17:
## dry-run

* `kubectl create deployment web --image=nginx -o yaml > .\section-16\web.yaml`
* `kubectl apply -f .\section-16\web.yaml --dry-run --valiate=false -o yaml`
* can see `replicas`
```
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: web
```
* `--dry-run=server`
* `kubectl apply -f .\section-16\web.yaml --dry-run=server --validate=false -o yaml`
* this does everything except write to **etcd**
* api strips out unknown bits
```
spec:
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: web
```

## kubectl diff

* `curl -O https://k8smastery.com/just-a-pod.yaml`
* `kubectl apply -f just-a-pod.yaml`
* edit `just-a-pod.yaml` to modify version of image0
* `kubectl diff -f just-a-pod.yaml`
```
-  - image: nginx
+  - image: nginx:1.17
```
* `kubectl delete -f just-a-pod.yaml`

# Section 18 - Rolling updates
* creates multiple replica sets
* each replica set has identical Pods
* two paramaters determine pace of rollout: `mexUnavailable` and `maxSurge`
* can be ablsolute or percent

* `kubectl apply -f https://k8smastery.com/dockercoins.yaml`
* `kubectl set image deploy worker worker=dockercoins/worker:v0.3`
* `kubectl rollout status deploy worker`
```
Waiting for deployment "worker" rollout to finish: 5 out of 10 new replicas have been updated...
```
* `kubectl get pods`
```
worker-578fcd5889-zdds6      1/1     Running            3 (5h8m ago)    3d4h
worker-8659b99d6-4hv4r       0/1     ImagePullBackOff   0               4m27s
worker-8659b99d6-656qs       0/1     ImagePullBackOff   0               4m27s
worker-8659b99d6-6ts9b       0/1     ImagePullBackOff   0               4m27s
worker-8659b99d6-8rbc7       0/1     ImagePullBackOff   0               4m27s
worker-8659b99d6-nkcx7       0/1     ImagePullBackOff   0               4m27s
```
   * MaxUnavailable=25%
   * so rollout terminated 2 replicas out of 10
   * MaxSurge=25%
   * so addition to replacing 2 replicas, the rollout also starting 3 more

## Recovering from failed deplpoyments
* `kubectl describe deploy worker`
```
Replicas:               10 desired | 5 updated | 13 total | 8 available | 5 unavailable
```
* `kubectl rollout undo deploy worker`
* `kubectl rollout status deploy worker`
* **Note undo only flips to the previous state**

# Rollout history and patching
* `kubectl rollout history deployment worker`
* we don't see all revisions (due to undo)
## Annotations
* `kubectl describe replicasets -l app=worker | grep -A3 Annotations`

## Rollback
* `kubectl rollout undo deployment worker --to-revision=1`

## Creating a yaml patch
If we need to change multiple things in the deployment
* we could use `kubectl edit deployment worker`
* but we cold also use `kubectl patch`
```
kubectl patch deployment worker -p "
spec:
   template:
      spec:
         containers:
         - name: worker
           image: dockercoins/worker:v0.1
   strategy:
      rollingUpdate:
         maxUnavailable: 0
         maxSurge: 1
   minReadySeconds: 10
"
```
# Section 20: Health Checks
* Health checks are **per** container
* they are probes apply to containers
* take action
* 3 optional probes:
   * liveness - dead or alive. **should only focus on this container**
   * readiness - ready to serve traffic
   * startup - stills starting up
* Different handlers: HTTP, TCP, programme execution

## Liveness probe
* if the liveness probe fails, the container is killed
* what happens next depends on the `restartPolicy` 
   * `Never`: the container is not restarted
   * `OnFailure` or `Always`: the container is restarted

## Readiness probe
* multicontainer checks
* ready to serve traffic
* if it "unready" is might become "ready" again
* To indicate temporary failure or unavailability
   * application can only service N parallel connections
   * runtime is busy doing garabage collection or initial data load

## Startup probe
* when container startup time can vary a lot
* will enable other probes when this is good

## Benefits of using probes 
* rolling updates proceed when containers actually ready

## Types
* keep them simple
* exec most expensive

* HTTP 
   * specicify URL andoptional headres
   * any 200 and 399 indicates success
* TCP Connection
   * the probe succeeds if the TCP port is open
*  Arbitary exec
   * a command is executed in the container
   * exit status of zero indicates success

## Timing and thresholds
* periodSeconds
* timeoutSeconds
* successThreshold
* failureThreshold
* initialDelaySeconds

## Proper healthcheck usage
* should probes check container dependencies
* healthcheck for workers
   * HTTP server is expensive
   * app touches a file and we check the timestamp is in range
   * check logs

## Questions
* do you want liveness, readiness or both (could use same with diff timing
* do we have existing HTTP
* do we need to add new endpoints
* are our healtchecks likely to use resource and slow down the app?
** do we need to depend on additional services (for readiness)

# Section 21: updating apps with probes and testing
* do we get temporary, recoverable glitches
   * then we use readiness
* do we get hard lock-ups requiring a restart?
   * then use liveness
* We don't know yet for Dockercoins
* Let's pick liveness

## Do we have HTTP endpoings
* 3 web services (hasher, rng, webui)

![endpoints](section-21/health.png)

* update `rng-deployment.yaml`
```
    spec:
      containers:
      - image: dockercoins/rng:v0.1
        name: rng
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 5
```
* `kubectl apply -f .`

## Testing the liveness probe
* `kubectl get pods -w`
* `kubectl get events -w`
* `kubectl attach --namespace=shpod -it shpod`
   * `CLUSTER_IP=$(kubectl get svc rng -o go-template='{{.spec.clusterIP}}')`
   * (get cluster ip)
   `ab -c 10 -n 1000 http://$CLUSTER_IP/1
## Better healthchecks
* Make sure healthcheck doesn't trip when performance is degraded due to external pressure
* using a readiness check would have fewer effects (but still imperfect)
* a possible combination:
   * readiness with a short timeout / low failure threshold
   * liveness check with a longer timeout / higher failure threshold

## Exec liveness check
* liveness is enough
* readiness is no use when there are not multiple containers
* redis = `redis-cli ping`

## Zombies
* when using `exec probes` we should make sure we have a *zombie reaper*
* we have the potential of creating zombies
* we can add [tini](https://github.com/krallin/tini) tp our images <- recommended
* or share the PID namespace between containers 