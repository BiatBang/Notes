# Kubernetes in Action

## Overview
### Container
Containers make usages of *Linux Namespace* (limits the scope of processes) and *Linux Control Groups* (limits the resource usage of processes) to make the separation. This is why windows suffers from not able to use Docker.
**Terms for docker**
- Image: a packaged program with metadata (env info).
- Registry: docker's repo. push to registry or pull from registry. 
- Container: a linux container, with limited resource.

## Kubernetes
One pods can have multiple containers (if necessary); one container should only have one process
`kubectl get po ... -o yaml | json` get detail with a yaml / json format
**pod yaml**
- **metadata**: name, namespace, labels, ...
- **spec**: actual description of the pod's contents, including containers, volumes, ...
`kubectl explain pods | pod.spec` print the documentation of pods and spec, sweet!
`kubectl logs <pod_name> [-c <container_name>]` print the log. with an 's'.
`kubectl port-forward <pod_name> <local_port>:<pod_port>` forwarding, like what's in the mailing system. Use a local port forward to handle the remote port.
**labels**: 2 kinds of labels
- app: define the pod's app, component, microservice (app=ui, as, pc, sc os)
- rel: whether pod is stable, beta, or canary (rel=stable, beta, canary)
**Canary**: when deploy a new version of app, only let part of the user access the new version, let others still hit the old version. 
`kubectl get po --show-labels` --show-labels option shows labels of pods
`kubectl label po <pod_name> <label_name>=<label_content> [--overwrite]` manually create a label to a pod
`kubectl get po -l <label_name>=[!]<label_content>` select pods with the label
`kubectl get po -l [!]<label_name>` select pods with(out) the label name
`kubectl get po -L <label_name>` select pods with the label name
not only pods can be labeled, nodes can be labeled, too. nodes can have specific features such as GPU computation. 
`kubectl label node <node_name> gpu=true` labels the node gpu=true.
then, use nodeSelector option, we can schedule apps to specific nodes.
```
spec:
  nodeSelector:
    gpu: "true"
  containers:
  - image: ...
```
**annotations**: 
`kubectl annotate pod <pod_name> <annotation>`
**namespace**:
`kubectl get po -n | --namespace <namespace>`
`kubectl create namespace <namespace>`
in yaml:
```
apiVersion: v1
kind: Namespace
metadata:
  name: custom-namespace
```
to switch namespace:
`alias kcd='kubectl config set-context $(kubectl config current-context)' --namespace`
**delete**:
`kubectl delete po [-l <label_name>=<label_content>]` terminate all the containers inside a pod
`kubectl delete ns <ns_name>` delete the namespace and all pods under it
replicationController can regenerate pods when the last pod is deleted. 

## Controllers
### Health Check
**liveness prob**:
periodically check if the container is alive
To determine if the container is running correctly:
- see if HTTP request get a success HTTP response
- TCP connection to a port succeed
- run an arbitrary command to exec the container, if exit 0, succeed
```
containers
  ...
  livenessProbe:
    httpGet:
      path: /
      port: 8080
      initialDelaySeconds: 15 ## before the first probe, wait for 15s, for the system to initialized
```
avoid auth, keep it simple
**ReplicationController**:
A controller keeps pods running. Either a node fails or a pod fails, RC will notice. Not only notices decrease, also notices unusual increase. Each RC has a label selector (manages the pods with a label);a replica count, knows how many replicas it needs; a template pod, knows what pod to create. 
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: ...
spec:
  replicas: 3
  selector: 
    app: ...
  template:
    metadata: 
      labels: 
        app: ...
    spec:
      containers:
      - name: ...
        image: ...
        ...
```  
`kubectl get rc` get rcs
RC can be edited at runtime. New replicas would use new template, and old will stay the same until modified. 
`kubectl edit rc <rc_name>` edit it at runtime, opens a new editor
`kubectl delete rc <rc_name> --cascade=false` cascade=false, then rc delete won't affect pods under it
**ReplicaSet**
Very similar to ReplicationController, but better. 
```
apiVersion: apps/v1beta2
kind: ReplicaSet
metadata:
  name: kube-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mkube
    template:
      metadata:
        labels:
          app: mkube
      spec:
        containers:
        - name: mkube
          image: gcr.io/google-containers/echoserver:1.10
```
selector.matchLabels have a more flexible way to identify pods with the label.
Even better with *matchExpressions*
```
selector:
  matchExpressions:
  - key: app
    operator: In
    values:
    - a
    - b
  - ...
```
a tuple, with key => name of the label, operator => {In, NotIn, Exists(without values), DoesNotExists}
**DaemonSet**
In the case of some app needs to be deployed on each pod, with only 1 container, DaemonSet comes in. DaemonSet only works when one node comes up. It's also ok to deploy this app on a subset of nodes.
```
apiVersion: apps/v1beta2
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        disk: ssd # this monitor will only be deployed on nodes with label "disk=ssd"
      containers:
      - name: main
        image: ....
```
Once the node gets the label, it'll run the daemonSet; once removed, the daemon will be terminated. 
**Job**
Deployment, DaemonSet, they both are long-term running applications, non-stop. If we want to run some containers who just run a short-time task, we need a Job. 
```
apiVersion: batch/v1
kind: Job
metadata: 
  name: batch-job
spec:
  template:
    metadata:
      labels:
        app: batch-job # job will be deployed on pods with the label
    spec:
      restartPolicy: onFailure # options: onFailure, Always, Never
      containers:
      - name: main
        image: ....
```
Multiple pods of Jobs can run both sequentially or simultaneously
```
spec:
  completions: 5 # five pods will run sequentially
  parallelism：2 # up tp 2 pods can run in parallel
```
parallel and sequential don't conflict with each other. It's like two pipelines, each one is one-task only. 
`kubectl scale job <job_name> --replicas 3` change the parallelism in the runtime
about failure:
```
spec:
  activeDeadlineSeconds: 5 # if the job is not done after 5s, it fails
  backoffLimit: 5 # if fails, 5 times to retry
```
**CronJob**:
A job needs to be done periodically
```
apiVersion: batch/v1beta1 # TBD
kink: CronJob
metadata:
  name: job-on-every-five-min
spec:
  schedule: "0,5,10,15 * * * *"
  startingDeadlineSeconds: 15 # after scheduled time (15s), if not started, fail
  jobTemplate:
    spec:
      template: 
        metadata: 
          labels: 
            app: 
        spec: 
          restartPolicy: OnFailure
          containers:
          - name: main
            image: ...
```
For Schedule, it's in the syntax of Linux Cron. 
`* * * * *` represent: minute, hour, day of month, month, day of week
`0,5,10,15 * * * *` means: in every hour (2nd *), every day of the month(3rd *), every month, (4th *), every day of the week(5th *), the 0, 5, 10, 15 min of every hour(1st *). That means: 0:00, 0:05, 0:10, 0:15, 1:00, 1:05, ...