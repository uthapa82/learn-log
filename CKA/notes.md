## Certified Kubernetes Administrator (CKA) 

**Docker Refresher** 

```bash
$ docker run ubuntu
$ docker ps 
$ docker ps -a
$ docker run kodekloud/simple-webapp
$ docker run -d kodekloud/simple-webapp 
$ docker attach name or id (a043d)

# run for 100 seconds 
$ docker run -d ubuntu sleep 100 

$ docker run -it centos bash 
# logs into the bash of container 

# remove docker containers 
$ docker rm container-name/id 

$ docker images 

# rmi for removing images 
$ docker rmi <image>

# count total images 
$ docker images -q | wc -l 

# tag 
$ docker run redis:latest

# i interactive docker run -i 
# -t sduo terminal , attach to terminal 
docker run -it 

docker run -p 80:5000 kodekloud/webapp

# map data from docker container to docker host , dockerHost:dockerContainer 
docker run -v /opt/datadir:/var/lib/mysql mysql 

# gives all the information 
docker inspect <name> 

# Container logs 
docker logs <containerID/Name>

# persist configuration data, we need to map a volume 
mkdir my-jenkins-data 
docker run -p 8080:8080 -v /root/my-jenkins-data:/var/jenkins_home -u root jenkins

#Run an instance of kodekloud/simple-webapp:blue and name the container blue-app, mapping port 8080 on the container to port 38282 on the host  -o HOST_Port:Conatiner_Port
docker run -p 38282:8080 --name blue-app kodekloud/simple-webapp:blue

docker history name test/simple-test-app

# create from dockerfile , failure cached 
docker build Docerfile -t test/my-test-app
docker build .

docker login 
# with tag 
docker build . -t account-name/my-simple-webapp:lite

```

- Creating own image 
    1. OS -Ubuntu
    2. Update apt repo - `apt-get update`
    3. Install dependencies using apt `apt-get install python`
    4. Install python dependencies using pip `pip install flask flask-mysql`
    5. Copy source code to /opt folder `copy . /opt/source-code`
    6. Run the web server using "flask" command `ENTRYPOINT FLAS_APP=/opt/source-code/app.py flask run`

    ```bash    
    docker build dockerfile -t test/my-custom-app
    docker push test/my-custom-app
    ```

- Docker file : ISTRUCTION ARGUMENT eg. FROM Ubuntu 
- Layered Architecture 


```bash
    $ docker ==> $ nerdctl 
    $ docker ps --filter ancestor=ubuntu --format '{{.ID}}'
    $ docker exec container-id sh -c 'command eg. echo "This is the file" >> /root/learning.txt'0
    $ docker run --name redis redis:alpine  ==> $ nerdctl run --name redis redis:alpine 
    $ docker run --name webserver -p 80:80 -d nginx ==> $ nerdctl run ---name webserver -p 80:80 -d nginx 
```

* Run docker without sudo

    ```bash
        bash# Add your user to the docker group
        sudo usermod -aG docker $USER

        # Apply the group change immediately (no logout needed)
        newgrp docker
    ```

* Environment Variables 

    ```bash
        docker run -e APP_COLOR=blue simple-webapp-color
        docker run -e APP_COLOR=green simple-webapp-color
        # example 
        docker run -e APP_COLOR=blue -p 38282:8080 --name blue-app kodekloud/simple-webapp
        
    ```

* Commands vs Entrypoint 
    - defines default command `CMD ["nginx"]` or `CMD ["mysqld"]`

    - append the command `docker run ubuntu [COMMAND]`

    - `docker run --entrypoint sleep2.0 ubuntu-sleeper 10`

* Docker Compose 
    - `docker run --link` to link services 

    - `docker compose up`

    - sample file 

        ```
        version: "3"

        services:
        redis:
            image: redis:alpine

        clickcounter:
            image: kodekloud/click-counter
            ports:
            - "8085:5000"
        ```

* Docker Engine / Docker Host 
    - Docker Deamon -> REST API -> Docker CLI 
    - Docker Deamon: Background process that manages Docker objects such as images, contaniners, volumes etc.
    - Docker CLI can be on same host or different, incase of remove host `docker -H=remote-docker-engine:2375 run nginx`

    - Namespace interProcess, Mount, Unix Timesharnig, Process ID, Network

    - Docker uses cgroups or control groups to restrict the amount of hardware resources allocated to each container.
        `docker run --cpus=.5 ubuntu` ==> ensures container does not take up more than 50 % of the host CPU at any given time 
        
        `docker run --memory=100m ubuntu` ==> limits the memory of container can use to 100 MB

* Docker Storage / File System
    - When we install docker in a system or host it creates following file structure 

        ```text
        /var/lib/docker
        ├── aufs
        ├── containers
        ├── images
        └── volumes
        ```
    - Layered architecture from top to bottom dockerfile, reuses the cached layers from another image or docker files 
    - Read Write Layer (container layer) above Read only (image layers)
    - COPY-ON-WRITE 
    
    - `docker run -v data_volume(docker on host):/var/lib/mysql (container volume)` old way, new way --> `docker run \ --mount type=bind, source=/data/mysql, target=/var/lib/mysql mysql`
     
    - volume mount and bind mount 

    - Storage drivers, automatically choosen based on the Operating System
        ```text
        - AUFS
        - ZFS
        - BTRFS
        - Device Mapper
        - Overlay
        - Overlay2
        ```
---
**Kubernetes**

```bash
    $ docker ==> $ nerdctl 
    $ docker run --name redis redis:alpine  ==> $ nerdctl run --name redis redis:alpine 
    $ docker run --name webserver -p 80:80 -d nginx ==> $ nerdctl run ---name webserver -p 80:80 -d nginx 
```

* CLI - crictl (cry control)

```bash
    $ crictl 
    $ crictl pull busybox 
    $ crictl images 
    $ crictl ps -a
    $ crictl exec -i -t xxxxxxxxxxxxxxxxx ls 
    $ crictl logs xxxxxxxxxxxxxxxxx
    $ crictl pods 
```
* default client that comes with ETCD is the ETCDCTL client. 

* create  key value pair `./etcdctl set/put key1 value1 `
* get value `./etcdctl1 get key1`
* `.etcdctl --version`
* `export ETCDCTL_API=3`

ETCDCTL is the CLI tool used to interact with ETCD.

ETCDCTL can interact with ETCD Server using 2 API versions - Version 2 and Version 3.  By default its set to use Version 2. Each version has different sets of commands.

For example ETCDCTL version 2 supports the following commands:

```
    etcdctl backup
    etcdctl cluster-health
    etcdctl mk
    etcdctl mkdir
    etcdctl set
```

Whereas the commands are different in version 3

```
    etcdctl snapshot save 
    etcdctl endpoint health
    etcdctl get
    etcdctl put
```

To set the right version of API set the environment variable ETCDCTL_API command

export ETCDCTL_API=3


When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don't work. When API version is set to version 3, version 2 commands listed above don't work.


Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don't worry if this looks complex:

```
    --cacert /etc/kubernetes/pki/etcd/ca.crt     
    --cert /etc/kubernetes/pki/etcd/server.crt     
    --key /etc/kubernetes/pki/etcd/server.key
```

So for the commands I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:

```
    kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key" 
```

* Lab1

    ```bash
    $ kubectl run --help

    # check the number of pods 
    $ kubectl get pods

    # create a new pod with the nginx image
    $ kubectl run nginx-pod --image=nginx

    # more details flag , check the node where the pods placed on 
    $ kubectl get pods -o wide 

    $ kubectl descibe pod <pod-name>

    $ kubectl get pod webapp -o jsonpath='{.spec.containers[*].image}'

    $ kubectl delete pod webapp

    # option 1 of creating pod 
    $ kubectl run redis --image=redis123 --dry-run=client -o yaml >> redis.yaml

    $ vi sample.yaml 
    $ cat sample.yaml 

    apiVersion: v1
    kind: Pod
    metadata:
      name: redis
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis

    $ kubectl apply -f sample.yaml 
    $ kubectl get pods
    ```

* Replicaset 

    ```bash
    $ kubectl get pods

    $ kubectl get replicaset

    $ kubectl get replicaset -o wide
    
    $ kubectl describe replicaset new-replica-set

    $ kubectl describe pod new-replica-set-712qw

    $ kubectl delete pod new-replica-set-2md6q 

    $ kubectl get pods
    
    $ kubectl explain replicaset

    $ vi replicaset-definition-1.yaml 
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: replicaset-1
    spec:
      replicas: 2
      selector:
        matchLabels:
          tier: frontend
      template:
        metadata:
          labels:
            tier: frontend
        spec:
          containers:
          - name: nginx
            image: nginx

    $ kubectl get replicationcontrollers 

    # to find the issue with the yaml file we can create and read the error 

    $ kubectl create -f replicaset-definition-1.yaml 

    $ vi replicaset-definition-2.yaml 
    apiVersion: apps/v1 ---> version need to match
    kind: ReplicaSet
    metadata:
      name: replicaset-2
    spec:
      replicas: 2
      selector:
        matchLabels:
          tier: frontend ---> labels need to match
      template:
        metadata:
          labels:
            tier: frontend ---> label need to match
        spec:
          containers:
          - name: nginx
            image: nginx
    

    $ kubectl create -f replicaset-definition-2.yaml 

    $ kubectl get rs

    $ kubectl delete replicaset replicaset-1 replicaset-2
    $ kubectl delete replicaset replicaset-1
    $ kubectl delete replicaset replicaset-2

    $ ls

    $ vi new-replica-set.yaml 

    $ kubectl edit rs new-replica-set

    $ kubectl get pods

    $ kubectl delete pod name1 name 2 name3..

    $ kubectl get pods

    # method-1
    $ kubectl scale rs new-replica-set --replicas=5

    # method-2
    $ kubectl edit rs new-replicaset --> change spec:--> replicas 

    $ kubectl scale -replicas=5 -f new-replica-set.yaml 

    $ kubectl scale --replicas=5 -f new-replica-set.yaml 

    $ vi new-replica-set.yaml 
    $ kubectl get pods
    $ kubectl replace -f new-replica-set.yaml 

    $ kubectl scale --replicas=2 -f  new-replica-set.yaml 

    ```

* Deployment 

    **Tip**
    As we might have seen already, it is a bit difficult to create and edit YAML files. Especially in the CLI. During the exam, you might find it difficult to copy and paste YAML files from browser to terminal. Using the kubectl run command can help in generating a YAML template. And sometimes, you can even get away with just the kubectl run command without having to create a YAML file at all. For example, if you were asked to create a pod or deployment with specific name and image you can simply run the kubectl run command.

    ```bash 
    Create an NGINX Pod

    kubectl run nginx --image=nginx

    Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

    kubectl run nginx --image=nginx --dry-run=client -o yaml

    Create a deployment

    kubectl create deployment --image=nginx nginx

    Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

    kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

    Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.

    kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

    Make necessary changes to the file (for example, adding more replicas) and then create the deployment.

    kubectl create -f nginx-deployment.yaml


    OR

    In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.

    kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml 

    ```

* Deployment Lab

    ```bash
    $ kubectl get deployment -o wide
    $ kubectl get rs 
    $ kubectl describe deployment frontend-deployment
    $ kubectl describe deployment frontend-deployment-cd6b557c-29npt
    $ kubectl describe pod frontend-deployment-cd6b557c-29npt
    $ ls
    $ vi deployment-definition-1.yaml 
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: deployment-1
    spec:
    replicas: 2
    selector:
        matchLabels:
        name: busybox-pod
    template:
        metadata:
        labels:
            name: busybox-pod
        spec:
        containers:
        - name: busybox-container
            image: busybox
            command:
            - sh
            - "-c"
            - echo Hello Kubernetes! && sleep 3600

    $ kubectl create -f deployment-definition-1.yaml 
    $ kubectl create deployment <name> --image=<image> --replicas=<number>
    $ kubectl get deploy
    $ kubectl create deployment --image=httpd:2.4-alpine httpd-frontend --dry-run=client -o yaml
    $ kubectl create deployment --image=httpd:2.4-alpine httpd-frontend --dry-run=client -o yaml > httpd-deployment.yaml
    $ cat httpd-deployment.yaml 
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    creationTimestamp: null
    labels:
        app: httpd-frontend
    name: httpd-frontend
    spec:
    replicas: 3
    selector:
        matchLabels:
        app: httpd-frontend
    strategy: {}
    template:
        metadata:
        creationTimestamp: null
        labels:
            app: httpd-frontend
        spec:
        containers:
        - image: httpd:2.4-alpine
            name: httpd
            resources: {}
    status: {}
                 
    $ vi httpd-deployment.yaml 

    $ kubectl create -f httpd-deployment.yaml 
    ```

* Services 

    ```bash
    $ kubectl get services or svc 
    NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   13m

    $ kubectl describe service kubernetes 

    $ kubectl get deployment

    $ kubectl get deployment -o wide

    $ ls
    $ vi service-definition-1.yaml 
        ---
        apiVersion: v1
        kind: Service
        metadata:
        name: webapp-service 
        namespace: default
        spec:
        ports:
        - nodePort: 30080
            port: 8080
            targetPort: 8080 
        selector:
            name: simple-webapp
        type: NodePort

    $ kubectl create -f service-definition-1.yaml 
    ```

* Namespace 

    ```bash
    $ kubectl get namespaces

    $ kubectl get pods --namespace=research

    $ kubectl run redis --image=redis --namespace=finance # -n=finance shortcut

    $ kubectl get pod --namespace=finance

    $ kubectl get namespaces 

    $ kubectl get pods --all-namespaces | grep blue

    $ kubectl describe pods blue

    # dns for blue pod in dev/marketing namespace , If it's in same namespace 

    kubectl get svs -n=marketing
    db-service.dev.svc.cluster.local
    db-service.marketing.svc.cluster.local

    # to make the default namespace a s dev
    $ kubectl config set-context $(kubectl config current-context) --namespace=dev

    apiVersion: v1
    kind: Namespace
    metadata:
      name: dev
      labels:
        app: myapp
        type: front-end

    ```