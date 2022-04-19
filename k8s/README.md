# CORE CONCEPTS
## Cluster architecture
Master: Manages, Plan, Schedule, Monitor Nodes
- ETCD Key value DB
- kube-scheduler: put workloads in a node
- Controllers (controller-manager, node-controller, replication-controller)
- kube-apiserver: responsible for all operations within the cluster
Worker Nodes: Host application as containers
- kubelet: manages the nodes
- kube-proxy: manages networking
## ETCD
kubeadm systems: 

```console
  kubectl exec etcd-master -n kube-system -- sh -c \
    "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 \
      --cacert /etc/kubernetes/pki/etcd/ca.crt \
      --cert /etc/kubernetes/pki/etcd/server.crt  \
      --key /etc/kubernetes/pki/etcd/server.key"
```

non kubeadm systems: 

```console
  ETCDCTL_API=3 etcdctl get / \
    --prefix --keys-only --limit=10 \
    --cacert /etc/kubernetes/pki/etcd/ca.crt \ 
    --cert /etc/kubernetes/pki/etcd/server.crt \
    --key /etc/kubernetes/pki/etcd/server.key
```

## kube-apiserver

kubeadm systems: `cat /etc/kubernetes/manifests/kube-apiserver.yaml`

non kubeadm systems: `cat /etc/systemd/system/kube-apiserver.service`

## kube-controller-manager
watch status of the nodes, remediate situation, node monitor period 5s. Node monitor grace period 40s

kubeadm system: `cat /etc/kubenernetes/manifests/kube-controller-manager.yaml`

non kubeadm systems: `cat /etc/systemd/system/kube-controller-manager.service`

## kube-scheduler
Decides which pod goes to which node. 

kubeadm systems: `cat /etc/kubenernetes/manifests/kube-scheduler`

## kubelet

Register node, create pods, monitor node and pods

## kube-proxy

Looks for new services and creates the appropiate network rules. Deployed as a daemon set

## Pods, ReplicaSets, Deployments

Replication controllers are replaced by replica sets

replication controller api: `v1`. 

replicaset api: `apps/v1`

selector field is the main difference with replication controller. replicaset can also take pods that are not created by the rs itself

`$ kubectl replace -f <yaml-file>`

`$ kubectl scale --replicas=6 -f <yaml-file>`

`$ kubectl scale --replicas=6 replicaset myapp-replicaset`

## Deployment

See https://kubernetes.io/docs/reference/kubectl/conventions/

## Namespaces

`<service-name>.<namespace>.svc.cluster.local`

To set a specific namespace

`$ kubectl config set-context $(kubectl config current-context) --namespace=dev`

To create a resource quota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard: 
    pods: "10"
    request.cpu: "4"
    request.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

## Services
A service is like a virtual server in the node. NodePort is the port from the node itself. Port is the port in the Service (virtual server) and targetPort is the port in the Pod. NodePort rage: 30000-32767

### NodePort
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
  - targetPort: 80
    port: 80
    nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

### ClusterIP
```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
  - targetPort: 80
    port: 80
  selector: 
    app: myapp
    type: back-end
```

## Imperative-declarative

`$ kubectl edit deployment nginx`

`$ kubectl replace -f nginx.yaml`

`$ kubectl replace --force -f nginx.yaml`

`$ kubectl apply -f .`

Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379 (This will automatically use the pod's labels as selectors)

`$ kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`

Or This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service

`$ kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml`

Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes. This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.

`$ kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`

Or this will not use the pods labels as selectors

`$ kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the kubectl expose command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

`$ kubectl expose pod redis --port=6379 --name redis-service`

Create a new pod called custom-nginx using the nginx image and expose it on container port 8080

`$ kubectl run custom-nginx --image=nginx --port=8080`

Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd). The target port for the service should be 80. Try to do this with as few steps as possible.

`$ kubectl run httpd --image=httpd:alpine --port=80 --expose`

# SCHEDULING

Taint: *person*. Bug: *toleration level*

`DNS SRV query _name._tcp.my-svc.my-ns`

To filter by selectors

`$ kubectl get pods --selector env=dev --no-headers | wc -l`

Node Affinity

`kubectl set pod-selector key=value`

`$ kubectl taint nodes <node-name> key=value:TaintEffect (NoSchedule | PreferNoSchedule | NoExecute)`

So, add taint:

`$ kubectl taint node node01 spray=mortein:NoSchedule`

Remove taint:
`$ kubectl taint node node01 spray=mortein:NoSchedule-+ 

In YAML (pod)

```yaml
nodeSelector:
  label_key: label_value
```
Resource request for a container
- Default 0.5 CPU 256 Mi
- In YAML (pod)

```yaml
  resources:
    request: 
        memory: "1Gi"
        cpu: 1
    limits:
        memory: "2Gi"
        cpu: 2
```

Add label to a node

`$ kubectl label node node01 color=blue`

Resource limitation for containers 
```yaml
  resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
An easy way to create a DaemonSet is to first generate a YAML file for a deployment, for instance:

`kubectl create deployment elasticsearch --image=k8s.gcr.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml > fluentd.yaml `

Next, remove the replicas, strategy and status fields from the YAML file using a text editor. Also, change the kind from Deployment to DaemonSet.

Finally, create the Daemonset by running 

`kubectl create -f fluentd.yaml`

staticPods (Created directly by kubelet)
```console
- kubelet ExecStart = 
    --pod-manifest-parth= ... 
      ... or ...
    --config=kubeconfig.yaml (key staticPodPath
```
- to check config 

`$ ps -aux | grep kubelet`

# LOGGING AND MONITORING

## deploy metrics server 

Download: `git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git`

Install: `cd kub<TAB> && kubectl create -f .`

## Commands

`kubectl top node`

`kubectl top pod`

## Logs - Docker
To see logs in the container (attach at run) 

`$ docker run -d <container-image>`

(container)

`$ docker logs -f ecf`

## Logs kubernetes
 
`$ kubectl logs -f <pod-name>`
  
 if there are more than one container
  
`$ kubectl logs -f <pod-name> <container-name>`

# APPLICATION LIFECYCLE MANAGEMENT

## Rollout command (status)

`$ kubectl rollout status deployment/<deployment-name>`

(history)

`$ kubectl rollout history deployment/<deployment-name>`

## Deployment Strategy
- Recreate or Rolling(default)
  - YAML (deployment)
    strategy: 
      ...
      type: Recreate | RollingUpdate
## Create new deployment (kubectl apply at yaml or set)
$ kubectl set image deployment/<deployment-name> <image-name>=<image>
## Rollback

$ kubectl rollout undo deployment/<deployment-name>
## Commands and arguments

```docker
FROM ubuntu
CMD sleep 5   ==> docker run ubuntu-sleeper sleep 10 
```
  
..or..

```docker
FROM ubuntu
CMD ["sleep", "5"]
```
  
FROM ubuntu
ENTRYPOINT ["sleep"]  ==> docker run ubuntu-sleeper 10

  with default value:
  FROM ubuntu
  ENTRYPOINT ["sleep"]
  CMD ["5"] ==> override >> docker run --entrypoint sleep2.0 ubuntu-sleeper 10

  (POD) (override CMD in dockerfile)
  spec: 
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      args: ["10"]
      command: ["sleep 2.0"]
## Configure environment variables 

Add environment variable. In the container spec
spec:
  container:
    ...
    env:
      - name: APP_COLOR   ==> (docker equivalent) >> docker run -e APP_COLOR=pink simple-webapp-color
        value: pink
From configmap
    env:
      - name: APP_COLOR
        valueFrom: 
          configMapKeyRef:
From secrets
    env:
      - name: APP_COLOR
        valueFrom: 
          secretKeyRef  :
## Configuring ConfigMaps
  1. Create ConfigMap
  2. Inject into Pod

  spec:
    containers:
    - name:
      ...
      envFrom:
      - configMapRef:
          name: app-config
Creation 
- imperative
  $ kubectl create configmap <config-name> --from-literal=<key>=<value>
  $ kubectl create configmap app-config --from-literal=APP-COLOR=blue --from-literal=APP_MOD=prod

  $ kubectl create configmap <config-name> --from-file=<path-to-file>
- declarative
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-config
  data:
    APP_COLOR: blue
    APP_MODE: prod
## Configure secrets
1. Create Secrets
2. Inject into Pod

Creation
- imperative
  $ kubectl create secret generic <secret-name> --from-literal=<key>=<value>
  $ kubectl create secret generic app-secret --from-literal=DB_Host=mysql

  $ kubectl create secret generic <secret-name> --from-file=<path-to-file>
- declarative
  Secrets has to be declared in base64
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-config
  data:
    APP_COLOR: <base64>
    APP_MODE: <base64>

  $ kubectl create -f secret-data.yaml
  To encode
  $ echo -n 'mysql' | base64
  
  spec:
    containers:
    - name:
      ...
      envFrom:
      - secretRef:
          name: app-secret
  Enviroment variables
  envFrom:
    - secretRef:
      name: app-config

  Single env variable
  env: 
    - name: <name_env_var>
      valueFrom: 
        secretKeyRef:
          name: <secret-name>
          key: <secret-key>
  
  Volume
  volumes: 
  - name: app-secret-volume
    secret: 
      secretName: app-secret

  Secrets in PODs and volumes
  ls /opt/app-secret-volumes
  cat /opt/app-secret-volumes/DB_Password

## Scale applications
 For executing commands inside a container
 $kubectl exec app -n elastic-stack -it -- cat /log/app.log

## Multi container pods
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
      - containerPort: 8080
  - name: log-agent
    image: log-agent
## Init containers

# CLUSTER MAINTENANCE
## OS upgrades
Pod Eviction timeout kube-controller-manager --pod-eviction-timeout. Time for a pod to be rescheduled to another node after its original one goes down. 
$ kubectl drain <node> 
==> For ignoring DaemonSets $ kubectl drain node01 --ignore-daemonsets
==> Force draining (A single node that is not part of a replica set) $ kubectl drain node01 --force
-- Its cordon
perform update
$ kubectl uncordon <node>
New pods cannot be scheduled to the node
$ kubectl cordon <node>
## k8s sw versions & Cluster Upgrade 
Component can be a different sw versions. 
kube-apiserver is the primary component. 
None of the other components should have a higher version
Controller manager and kube-scheduler can be a one version lower (X-1)
kubelet and kube-proxy can be a two versions lower (X-2)
kubectl can be one higher or lower than the kube-apiserver (X+1 > X-1)
How to upgrade
- Cloud provider: easy
- kubeadm
  $ kubeadm upgrade plan
  $ kubeadm upgrade apply
- the hard way
For upgrade
master$ apt-get upgrade -y kubeadm=1.12.0-00
master$ kubeadm upgrade apply v1.12.0
Kubelet has to be updated 
master$ apt-get upgrade -y kubelet=1.12.0-00
master$ systemctl restart kubelet

master$ kubectl drain node-1
node-1$ apt-get upgrade -y kubeadm=1.12.0-00
node-1$ apt-get upgrade -y kubelet=1.12.0-00
node-1$ kubeadm upgrade node config --kubelet-version v1.12.0
node-1$ systemctl resetart kubelet
master$ kubectl uncordon node-1
## Backup and restore methods (ECTD)
When etcd starts 
ExecStart=/usr/local/bin/etcd \\
  ..
  --data-dir=/var/lib/etcd <<-- here is the data located.
- Snapshot solution for etcd
 $ ETCDCTL_API=3 etcdctl snapshot save snapshot.db
- Status
 $ ETCDCTL_API=3 etcdctl snapshot status snapshot.db
- Restore
  (1. stop the api-server)
  $ service kube-apiserver stop 
  (2. restore the snapshot)
  $ ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup
  (3. use the new datadiectory in etcd.service)
  (4. reload daemon)
  systemctl daemon-reload
  (5. restart the service) 
  service etcd restart
  BY ALL COMMANDS, PROVIDE ENDPOINTS, CACERT CERT AND KEY FOR ETCD

# SECURITY
Accounts: User and Service. We cannot create users but service accounts
$ kubectl create serviceaccount sa1
$ kubectl get serviceaccount

Auth mechanisms: Static Password File, Static Token file, Certificates or Identity Services
- Static pass file
  Exec=...kube-apiserver \\
    .. 
    --basic-auth-file=user-details.csv
- Static Token file
  Exec=...kube-apiserver \\
    .. 
    --token-auth-file=user-details.csv
## TLS in k8s
Generate Keys openssl 
  (CA)
  $ openssl genrsa -out ca.key 2048
  (OTHER e.g. admin user)
  $ openssl genrsa -out admin.key 2048
Certificate Signing Request
  (CA)
  $ openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr 
  (OTHER e.g. admin user)
  $ openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr
Sign Certificates
  (CA)
  $ openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
  (OTHER e.g. admin user)
  $ openssl x509 -req -in admin.csr -CA ca.crt -CAKey ca.key -out admin.crt
## View certificate details
  $ openssl x509 -in <path-to-cert/cert-name.crt> -text -noout
  (To see errors in logs. etcd as service)
  $ journalctl -u etcd.service -l
  (To see errors in logs. etcd as pod)
  $ kubectl logs etcd-master
  $ docker logs <container-id>
## Certificates API
  CA cert manages the security by signing CSRs. 
  1. Create CSR object
    $ openssl genrsa -out jane.key 2048
    $ openssl -new -key jane.key -subj "/CN=jane" -out jane.csr
    (Create yaml file with base64 encoded CSR)
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: jane
    spec:
      groups:
      - system:authenticated
      usages:
      - digital signature
      - key encipherment
      - server auth
      signerName: kubernetes.io/kube-apiserver-client
      request: <BASE64 of CSR>
    (for base64)
    $ cat jane.csr | openssl enc -A -base64    
  2. Review Requests
    $ kubectl get csr
  3. Approve Requests
    $ kubectl certificate approve jane
  4. Share Certs to Users
  (Deny an agent request)
  $ kubectl certificate deny agent-smith
  (Delete a certificate signing request)
  $ kubectl delete csr agent-smith
## KubeConfig
  (Example of API query)
  $ curl https://my-kubernetes-cluster:6443/api/v1/pods \
     --key admin.key
     --cert admin.crt
     --cacert ca.crt
  (with kubectl)
  $ kubectl get pods
     --server my-kubernetes-cluster:6443
     --client-key admin.key
     --client-certificate admin.crt
     --certificate-authority ca.crt
  All that info goes to $HOME/.kube/config
  KubeConfig has three sections: Clusters, Contexts, Users (Cluster: google, User:dev, context: dev@google)
  >>
  apiVersion: v1
  kind: Config
  current-context: my-cluster-admin@my-cluster
  clusters:
  - name: my-cluster
    cluster:
      certificate-authority: ca.crt
      server: https://my-cluster:6443
  contexts:
  - name: my-cluster-admin@my-cluster
    context:
      cluster: my-cluster
      user: my-cluster-admin
      namespace: my-namespace
  users:
  - name: my-cluster-admin
    user:
      client-certificate: admin.crt
      client-key: admin.key
  <<EOF
  
  To see the configuration
  $ kubectl config view
  To get the current context
  $ kubectl config current-context --kubeconfig=<path-to-kubeconfig>
  To set other context
  $ kubectl config use-context <context-name> --kubeconfig=<path-to-kubeconfig>

## API Groups, Authorization, Role Based Access Controls
APIs /metrics, /healthz, /version, /api, /apis, /logs
  core => api/ v1 => pods, pvc, serviceaccounts, etc
  named => apis/ => /apps, /extensions, /networking.k8s.io, /storage.k8s.io, /authentication.k8s.io, /certificates.k8s.io
$ curl http://localhost:6443 -k
(Start a proxy which uses the credentials in kubeconfig)
$ kubectl proxy 

Authorization: Node, ABAC, RBAC, Webhook. To set the auth, in ExecStart of kube-apiserver=> --authorization-mode=Node,RBAC,Webhook
Create a RBAC (developer-role.yaml)
1. Create a Role definition
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules: 
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
2. Create a Role binding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io

$ kubectl get roles
$ kubectl get rolebindings
$ kubectl describe role developer
- Check Access: can-i
$ kubectl auth can-i create deployments
$ kubectl auth can-i delete nodes
- Check Access: impersonation
$ kubectl auth can-i create deployments --as dev-user
$ kubectl auth can-i delete nodes --as dev-user
$ kubectl auth can-i delete nodes --as dev-user --namespace test
$ kubectl auth can-i watch pod/dark-blue-app --namespace blue --as dev-user 

## Docker registry
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred

## Cluser Role and Role bindings
example namespaces resources: pods, replicasets, jobs, deployments, services, secrets, roles, rolebindings, configmaps, pvc
example non-namespaces resources: nodes, pv, clusterroles, clusterrolebindings, certificatesigningrequests, namespaces

1. Create Cluste Role
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules: 
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]

2. Create ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io

## Service Accounts
Used by apps. To create
$ kubectl create serviceaccount <name>
Creates a token automatically. It's stored as a secret object. To view the token (Stored in a secret object)
$ kubectl describe secret dashboard-sa-token-kbbdm
If the app is within the cluster, the secret can be mount within the pod. A default service account is always created for every namespace
We can establish pod service account with serviceAccountName: key

## Image Security
Image structure access: Registry / Username / Image
Create docker credentials
$ kubectl create secret docker-registry regcred \
    --docker-server= private-regist \
    --docker-username=  \
    --docker-password=  \
    --docker-email=     \
In pod definition spec: key imagePullSecrets: -name: regcred
## Security Contexts
Settings in the container will override settings in the pod
spec:
  securityContext:
    runAsUser: 1000
.. for capabilities..
spec:
  containers:
    ..
    securityContext:
      runAsUser: 1000
      capabilities:
        add: ["MAC_ADMIN"]
## Network policy
A network policy is an object in k8s. 
Example ingress policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  policyTypes:
  - Ingress
  ingress:
  - from: 
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
Network solution should support network policy
selectors: podSelector and namespaceSelector (matchLabels), ipBlock (cidr). For instance
ingress:
- from: 
  - podSelector:
      matchLabels:
        name: api-pod
    namespaceSelector:
      matchLabels:
        name: prod
  - ipBlock:
      cidr: 192.168.5.10/32

# STORAGE
## Storage in Docker
Docker install its data on /var/lib/docker (aufs, containers, image, volumes)
## Volumes
apiVersion: v1
kind:Pod
...
spec:
  containers:
  - image: alpine
  ...
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  ...
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory

  Volume options: NFS, GlusterFS, Flocker, ceph, SCALEIO, AWS EBS Azure, GCP. For instance
    volumes:
    - name: data-volume
      awsElasticBlockStore:
        volumeID: <volume-id>
        fsType: ext4
## Persistent volumes
With PV a pool of storage can be defined and can be required by Persistent Volume Claims.
AccessModes: ReadOnlyMany, ReadWriteOnce, ReadWriteMany
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
  persistentVolumeReclaimPolicy: Recycle|Delete
## Persistent Volume Claims
k8s tries to find a Persistent Volume for the claim. There is a 1:1 relationship between PV and PVC. no other claim can use the remaining space available in a PV after being reclamed by a PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi  
## PVC in a Pod
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
## Storage Classes
With classes we can define dynamic provisioning of storage. A PV can be created automatically by the storage class
pvc-def.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  ..
  storageClassName: google-storage

sc-definition.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd

Storage Classes which make use of no-provisioner do not support dynamic provisioning.

# NETWORK
## Switching route
ip link
ip addr
ip addr add 192.168.1.10/24 dev eth0
ip route
To show IP routing table
$ route
To add a router on 192.168.1.1 which knows 192.168.2.0
ip route add 192.168.2.0/24 via 192.168.1.1
By default linux does not forward packages from one interface to another. 
cat /proc/sys/net/ipv4/ip_forward
## DNS
$ cat /etc/resolv.conf
> nameserver 192.168.1.10
Host first looks at local /etc/hosts file
Defined in 
$ cat /etc/nsswitch.conf
> hosts: files dns
search in cat >> /etc/resolv.conf
...
search mycompany.com
$ nslookup 
does not consider local entries. 
dig www.google.com
## Network namespaces
To create a new network namespace
$ ip netns add red
To list
$ ip netns
ip link inside the red namespace
$ ip netns exec red ip link
$ ip -n red link
The same for routing and arp
$ arp
$ ip netns exec red arp 
$ route
$ ip netns exec red route
Add network connectivity with a virtual network
$ ip link add veth-red type veth peer name veth blue
$ ip link set veth-red netns red
$ ip link set veth-blue netns blue
Assign ip address
$ ip -n red addr add 192.168.15.1 dev veth-red
$ ip -n blue addr add 192.168.15.2 dev veth-blue
$ ip -n red link set veth-red up
$ ip -n blue link set veth-blue up 
It should work now for exec in the namespace
Virtual Bridge: Linux Bridge or OvS (Open vSwitch)
$ ip link add v-net-0 type brigde
$ ip link set dev vn-net-0 up 
We delete teh virtual cables and connect them to the bridge
$ ip link set veth-red netns red
$ ip link set veth-red-br master v-net-0
$ ip link set veth-blue netns blue
$ ip lin k set veth-blue-br master v-net-0
$ ip -n red addr add 192.168.15.1 dev veth-red
$ ip -n blue addr add 192.168.15.2 dev veth-blue
$ ip -n red link set veth-red up
$ ip -n blue link set veth-blue up
To make it possible the private network reach the local one, add a route
$ ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5
Add NAT functionality to the host
$ iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE
Reach Internet from namespace 
$ ip netns exec blue ip route add default via 192.168.15.5
Connectivity from outside with NAT
$ iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT
## Docker networking
Docker creates a network interface in the host
$ docker network ls
$ ip link
(returns 172.17.0.1 at docker0)
Docker also creates a namespace 
$ ip netns
## Container Network Interface
$ bridge add <id> /var/run/netns/<id>
## Cluster networking
on master
ETCD 2379
kube-api 6443
kubelet 10250
kube-scheduler 10251
kube-controller-manager 10252
on nodes
services 30000-32767
kubelet 10250
## Test 1
Internal IP: kubectl get nodes -o wide
## Pod Networking
Every pod should have an IP address
Every pod should be able to communicate with every other POD in the same node
Every pod should be able to communicate with every other pod on other nodes without NAT
Many networking solutions: weaveworks, flannel, ciclium, vmware nxs, 
Kubelet executes script for add and remove veth
--cni-conf-dir=/etc/cni/net.d
--cni-bin-dir=/etc/cni/bin
./net-script.sh add <container> <namespace>
## CNI in k8s
## Service Networking
kube-proxy can have modes: userspace | iptables | ipvs
kube-apiserver specifies the service IP ranges in the switch --service-cluster-ip-range
To see iptables for a service db-service and logs
$ iptables -L -t nat | grep db-service
$ cat /var/log/kube-proxy.log
## DNS in kubernetes
Pods in the same namespace 
curl http://service-name.namespace.<root=cluster.local>
Record for pods
10-244-2-5.apps.pod.cluster.local
In every pod /etc/resolv.conf has an entry 10.96.0.10
CoreDNS is the recomended setup. 
cat /etc/coredns/Corefile
## Ingress
There are many ingress controllers: nginx, contour, haproxy, traefik, istio
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  backend:
    serviceName: wear-service
    servicePort: 80
--- 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-war-watch
spec:
  rules:
  - host: wear.my-online-store.com
    http: 
      paths:
      - backend:
          serviceName: wear-service
          servicePort: 80
  - host: watch.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: watch-service
          servicePort: 80
## Design and install a k8s cluster
HA for ETCD, api server. 

# TROUBLESHOOTING
## Apps 
See pod logs
$ kubectl logs web -f --previous
## Control plane node
$ kubectl get pods -n kube-system 
$ kubectl logs kube-apiserver-master -n kube-system
$ service (kubelet | kube-apiserver | kube-controller-manager) status
$ journalctl -u kube-apiserver
## Nodes
$ kubectl describe node worker-1
Check kubelet status
$ journalctl -u kubelet
Check certificates
openssl x509 -in /var/lib/kubelet/worker-1.crt

kubectl run test-nslookup2 --image=busybox:1.28 --rm -it --restart=Never -- nslookup 10-50-192-1.default.pod
