# Connecting to an existing cluster for devs

When trying to develop a new service, it makes sense to try to connect it as if it were part of an existing cluster. But this requires rebuilding a docker image, writing a template file and creating the deployment, possibly disrupting the cluster and breaking everything.

To solve this issue, we invite you to look into *telepresence* (https://github.com/datawire/telepresence).
Telepresence works by creating well crafted iptables rules, tied to a shell. Those rules will detect the cluster IP range and redirect every request sent to that range to a proxy that is instantiated cluster-side.
For DNS, the approach is to forward all dns requests back to the proxy, which will try to resolve them against the cluster dns. If it fails, it will instruct the client to redirect to one of the regular DNS.

> **Warning** Although telepresence can run docker images with the adjusted network, docker's `--init` argument is passed by default to the starting container. This creates a process before our entrypoint and gives it PID 1. Since our containers run PID 1, systemd fails to start and the container just hangs. To solve this issue, we must pass `--init=false` as an argument to telepresence when running. Doing so however is NOT guaranteed to work, as this essentially results in a call like `docker run --init ... --init=false ...`, which could fail depending on docker versions. Discussions are ongoing with datawire to change this behaviour.


## How to configure to connect to the cluster

Telepresence expects being able to call kubectl, for that to happen, the correct kubeconfig file needs to be passed to kubectl. Once this is done we can run `telepresence --run-shell` to get a shell "in the cluster".

With this shell, we can access cluster resources easily by using their Service names.
Some quick kubectl commands to get you running:

`kubectl get --all-namespaces svc` <- this will show all services running in all namespaces. The DNS exposes each services as "service_name.namespace.svc.cluster_name" so if I want to access the service "postgresql" in the default namespace, I can run `nslookup postgresql.default.svc.cloudtrust`. The dns will by default try the domains "default.svc.cloudtrust, svc.cloudtrust, and .cloudtrust" so this means we can also run `nslookup postgresql` to get the same result.
However, for services in another namespace, such as the dns itself, we need `nslookup kube-dns.kube-addons`.

To check the state of running deployments/pods, you can run `kubectl get deployments` or `kubectl get po`. For more information, there is also the `kubectl describe po` command (also works with deployments). If something seems wrong inside the cluster, `kubectl exec $POD_NAME` functions like a docker exec call, with similar options.
