
# Practice Exam for Certified Kubernetes Administrator (CKA) and Certified Kubernetes Application Developer (CKAD)

`Certified Kubernetes Administrator` (CKA) and `Certified Kubernetes Application Developer` (CKAD) exams are all about your practical experience with Kubernetes. 
The exam is quite intense as it requires two hours of concentrated effort during which you are asked to solve various problems. 

CKA exam not only requires knowledge of API primitives, but also has concepts related to the Kubernetes internals such as `etcd`, `tls bootstrap`, `kubelet`, etc.

> The purpose of the CKA program is to provide assurance that certificate holders have the skills, competency and knowledge required to perform responsibilities of Kubernetes administrators.

While CKAD exam is totally dedicated to practices such as `Deployments`, `Pods`, `DaemonSets` and other API primitives. 

> The CKAD exam certifies that certificate holders can design, build, configure, and expose native cloud applications for Kubernetes.

Some sample questions to pass CKA and CKAD exams are listed below:

* Create a pod named `web` using image `nginx:1.11.9-alpine` on ports `80` and `443` in the `salman` namespace and the `problem1` label.

```
$ kubectl run web --image=nginx:1.11.9-alpine --port=80 --port=443 -l question=problem1
```

OR

```
apiVersion: v1
kind: Pod
metadata:
  name: web
  namespace: salman
  labels:
    question: problem1
spec:
  containers:
  - name: web
    image: nginx:1.11.9-alpine
    ports:
    - containerPort: 80
      protocol: TCP
    - containerPort: 443
      protocol: TCP
```


* List all the pods in the `salman` namespace sorted by name.

```
$ kubectl get pods --sort-by=.metadata.name -n salman
```


* List all the pods in the `salman` namespace sorted by age.

```
$ kubectl get pods --sort-by=.metadata.creationTimestamp -n salman
```


* Create a persistent volume named `example-pv` and specification that defines a volume for `10Gi` mapping to hostpath `/opt/salman`. 

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 10Gi
  hostPath:
    path: /opt/salman
  asccessModes:
  - ReadWrireOnce
```


* Create a config map from file `/var/lib/kubelet/config.yaml` and create an `nginx` pod that sees the configmap as a volume in the same path within the container, assign port `80` and expose it as a service `webservice` as well. 

```
$ kubectl create configmap mariadb-config --from-file=/var/lib/kubelet/config.yaml

$ nano nginx-web.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
      volumeMounts:
      - mountPath: "/var/lib/kubelet/config.yaml"
        name: configmapvolume
  volumes:
  - name: configmapvolume
    configMap:
      name: mariadb-config
$ kubectl apply -f nginx-web.yaml

$ kubectl expose pod nginx --name=webservice
```


* Create a busybox container named `mybox` that sleeps `60000`.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: k8s.gcr.io/busybox
        args:
        - sleep
        - "60000"
```


* Attach liveness probe to the container and restart if the environment variable named `USER` is null or undefined.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: k8s.gcr.io/busybox
        args:
        - sleep
        - "60000"
        livenessProbe:
          exec:
            command:
            - "/bin/sh"
            - "-c"
            - "[[ ! -z $USER ]]"
          initialDelaySeconds: 5
          periodSeconds: 5
```


* Create a pod that is only scheduled on nodes with the `SSD` storage.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    storage: ssd
```


* Create a deployment running `nginx` version `1.12.2` that will run in 2 pods. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
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
        image: nginx:1.12.2
        ports:
        - containerPort: 80
```


* Scale this to 4 pods.

```
$ kubectl scale deployment nginx-deployment --replicas=4
```


* Scale it back to 2 pods.

```
$ kubectl scale deployment nginx-deployment --replicas=2
```


* Upgrade this to 1.13.8

```
$ kubectl set image deployment nginx-deployment nginx=nginx:1.13.8 --record
```


* Undo the upgrade.

```
$ kubectl rollout undo deployment nginx-deployment
```


* Create an nginx deployment which uses a scratch disk.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
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
        volumeMounts:
        - mountPath: /data
          name: cache-volume
      volumes:
      - name: cache-volume
        emptyDir: {}
```


* List all the pods showing name and namespace with a json path expression.

```
$ kubectl get pods -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.namespace}{"\n"}{end}' --all-namespaces
```
