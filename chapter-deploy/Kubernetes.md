# Kubernetes in Cloudtrust

Kubernetes' purpose is to define the cluster's desired state through yaml files, and then apply those files.
Essentially, this means specifying what containers to run, in how many replicas, where (to some extent). It allows us to treat a bunch of machines as a single one.
Kubernetes isn't inherently complex, but it has many different parts and a lot of small behaviour quirks, and it can feel overwhelming to a beginner. This section attempts to demistify all of the fluff of kubernetes that cloudtrust uses.

## Defining resources
Kubernetes understands all resource definition through yaml files. For a complete reference refer to : https://v1-7.docs.kubernetes.io/docs/api-reference/v1.7/

## Pods

Pods in kubernetes is simply a set of containers that share a few namespaces. Namely Network and PID by default.

> **Warning** In our case, where we run systemd in our containers, PID namespace sharing has to be disabled!! This is due to the fact that kubernetes runs a `pause` initialization container to initialize the common namespace for containers in the pod. This `pause` program gets PID 1 and systemd refuses to run!

Pods aren't meant to be manipulated directly. Just like we used to run container by `start`ing them in a systemd unit file, we want kubernetes to restart pods when they fail. So pods aren't used in cloudtrust, except for one category of pods.

## Static Pods

Remember that kubernetes has 5 components that run on a master node, and 2 that run on every slave. Among those, the kubelet and kube-proxy *have* to run on the host, as one runs containers and the other manages iptables rules. The 3 master components, however, can run inside containers. But running containers that won't be managed by kubernetes among a whole ecosystem of containers can be foolish (To understand that, simply run a `docker ps` on a kubernetes node, and try to find the containers you've run by hand). To this end, kubernetes provides a mechanism to run static pods. Static pods are kubernetes pods that are launched by the kubelet without interaction with the Apiserver. To define static pods, we have to provide a directory that the kubelet watches for changes and write manifests inside. This is a great way to run master components (apiserver, scheduler, controller-manager, etcd...) in containers and see them through kubernetes.

## Deployments

Kubernetes pods are ephemeral and not very useful on their own, as they can't scale up or down easily. The answer to this problem is deployments. Deployments allow us to specify a number of expected replicas, and come with nice rolling update features. When a deployment is created, it automatically spawns a replicaset controller, which specifies the desired state in the controller manager. This will ensure that X replicas are always running (if possible).

## Service

Once containers are spawned, it's important to make them accessible. One way to do so is to use a kubernetes Service. Services are basically a wrapper around pods that allow them to be exposed either through a ClusterIP or a NodePort. A ClusterIP service is just that, a Cluster IP that will be forwarded to one of the services' backend pod (note that this doesn't provide any real load balancing!!), while a NodePort is essentially a port on any node that will forward to one of the services backends (again, no load balancing there!). Services by themselves are pretty useless, but shine when used in conjunction with a cluster DNS.

## DNS

A cluster DNS is a pod with a fixed ClusterIP that provides name resolution for the cluster. It is composed of 4 parts:
 - Service: The server needs a known IP which is reachable inside the cluster both by nodes and by containers. Furthermore it needs to be deterministic. To this end, the service IP is a fixed Cluster IP, fixed to 10.254.0.10 (where 10.254.0.0/16 is the cluster IP range)
 - ServiceAccount: The DNS needs access to the kubernetes API, and uses a service account for that
 - ConfigMap: The DNS has a configmap available in case we need to change the configuration dynamically. It needs to be initialized to give upstream servers so that cluster components have regular name resolution as well as cluster.
 - Deployment: The DNS deployment is made of 3 different containers. One is the kube-dns container, that listens to the apiserver for new services. It dynamically updates its configuration and acts as a secondary DNS for the 2nd container: dnsmasq. A third container provides healthchecking for the pod. The only parameter required is by the kube-dns and is simply the cluster name.

## Ingress Controller

Once we have services and name resolution for services, pods within the cluster can communicate with one another, but nothing can come from the outside. This would typically be the job of a reverse proxy like nginx or haproxy, but those wouldn't be able to dynamically load configuration from the apiserver. This is where ingress controllers come in. An Ingress controller is basically a wrapper over an existing reverse proxy, that is also able to dynamically change its configuration. We use the nginx ingress controller.

## Ingress Rules

One a controller is defined, we need to feed it with rules regarding hostnames and what service they map to. This is the job of Ingress Rules.

> **Hint** To update an SSL certificate, we need to create a kubernetes secret of type "tls" using this syntax : `kubectl create secret tls foo-secret --key /tmp/tls.key --cert /tmp/tls.crt`

<!-- -->
> **Hint** To test an https connection with host based routing, we can't just use `curl -H "Host: servername" https://127.0.0.1/`. This is because of *SNI*, we need to inform the server what hostname we are trying to access directly in the SSL handshake. As such, we need to do this: `curl https://servername:443/ --resolve servername:443:127.0.0.1`. This has been the source of a lot of pain.

## Secrets

Kubernetes has the notion of secrets, which are volumes meant to contain sensible data like encryption keys, certificates, account keys etc... They can be mounted in pods, and encrypted at rest.

## Admission Controller

In kubernetes, admission controllers are controllers that try to authenticate calls to the kubernetes API. And some have deep ramifications in how things go on when they're defined. In our current setup, we only use one admission controller: the ServiceAccount controller. The ServiceAccount controller allows the administrator to define service accounts for pods to use when trying to access the kubernetes API. In particular, when this controller is defined, the apiserver automatically creates and mounts a default serviceaccount secret (per namespace) in every pod that is created, in `/var/run/kubernetes/` This allows services that need access to the kubernetes API (ie: the DNS) to access the kubernetes API.

## SSL

SSL is important when trying to do things securely. To this end, kubernetes allows communication to be in https rather than plain http. To this end we must define certificates to be used by the various components. In particular, we need 2 certificates : One that will act as a root CA that will be trusted on every node and pod  and one that will be the server's certificate. The root CA is mounted in every pod as a secret (thanks to the `--root-ca` flag of the controller manager) 

## Config Maps

Config maps are dynamic configuration that are mounted as files in pods.

## Daemon Sets

A daemonset is a kubernetes object that will ensure that each node runs a copy of a pod. This is very useful for ceph, or a daemonized ingress controller.

