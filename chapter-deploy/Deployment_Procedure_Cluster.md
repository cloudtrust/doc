# Deployment guide

This document explains how to deploy the CloudTrust components on a cluster.

Disclaimer: The whole deployment procedure was written by a single engineer new to this whole ecosystem, using many different conflicting open-source resources as inspiration.  
This was done on a best-effort basis, but lacks any sort of rigorous review by an experienced engineer.
As such, it is best to view this work with a critical eye, and try to improve upon it, even refactoring it deeply, if it is judged adequate.

## Minimal hardware requirements
- CPU: 4 vCPU
- RAM: 16 GiB
- Disk: 100 GiB

These requirements are for the deployment of the Cloudtrust solution with all the components.

One cluster requires at least 3 VMs.


## Prerequisites

Install the `Container Management` environment on your own machine:

    dnf install @'Container Management'


## Installation

We will use 2 Git repositories:
- The `on-premise-runtime` repository contains generic Ansible scripts.
This repository is hosted on GitHub, at https://github.com/cloudtrust/on-premise-runtime
- Another repository contains the configuration files, secrets and certificates required to install the Cloudtrust components on the cluster nodes.
The reference repository is the `dev-config` repository, hosted at https://github.com/cloudtrust/dev-config.
You must create a new repository with your own configuration based on this reference repository.


### Configuration

In this first part, we will set all the parameters that are specific to each environment.
To do so, we will work in the repository containing the configuration parameters for the cluster nodes.

Clone the repository:

    git clone <repo-url>

In the file `deployable/inventories/main.ini`, specify the IP addresses of your VMs, such as:

    [kube-master]
    <ip1>
    <ip2>

    [etcd:children]
    kube-master

    [kube-slave]
    <ip1>
    <ip2>

In the directory `deployable/roles/baseimage/keys/`, there are two files `id_ssh` and `id_ssh.pub`.
You need to specify an SSH key which gives access to a Github account.
This will be used to clone the cloudtrust repositories when building docker images.  
Note that for ELCA's deployments, we already have a service account setup in Github.
You can retrieve this key like so:

    git clone ssh://git@git.elcanet.local:7999/cloudtrust/devel-doc.git
    cd devel-doc/components/keys/
    gpg -d id_ssh.gpg > id_ssh
    # The password for decrypting this file is in the Keepass file
    # Then, copy the files `id_ssh` and `id_ssh.pub` into `<config-repo>/deployable/roles/baseimage/keys/`.

For the following steps, it is required to have a CA certificate to sign the certificates of each components.
If you do not already have one, you can generate your own self-signed CA certificate using OpenSSL.  
Note that for ELCA's deployments, we already have a PKI infrastructure in the repository `pki-config`.


- Ceph:
  Secret keys are located in `deployable/roles/ceph/files/`.
  If you need to generate new keys, you can use the following commands:

      # Make sure you have the Ceph tools installed. They can be installed with `dnf install ceph`.
      ceph-authtool --gen-print-key > deployable/roles/ceph/files/admin.key
      ceph-authtool --gen-print-key > deployable/roles/ceph/files/bootstrap.key
      ceph-authtool --gen-print-key > deployable/roles/ceph/files/mon.key
      ceph-authtool --gen-print-key > deployable/roles/ceph/files/user.key

  Alternatively, you can have the ansible playbooks generate new keys during each deployment by setting `dynamic_keys: True` in the file `deployable/roles/ceph/vars/main.yml`.

- etcd:
  Certificates are located in `deployable/roles/etcd-host/files/`.
  If you need to generate new certificates, you can use the `gen_certs.sh` script.
  You may need to adapt the `*.conf` files depending on your environment.

      cd deployable/roles/etcd-host/files/
      PKI_PATH="<path to the directory containing the ca.crt file>" ./gen_certs.sh

- Grafana:
  Certificates are located in `deployable/roles/grafana/files/`.
  If you need to generate new certificates, you can use the `gen_certs.sh` script.
  You may need to adapt the `*.conf` files depending on your environment.

      cd deployable/roles/grafana/files/
      PKI_PATH="<path to the directory containing the ca.crt file>" ./gen_certs.sh

- IDP test client:
  Certificates are located in `deployable/roles/idp-test-client/files/`.
  If you need to generate new certificates, you can use the `gen_certs.sh` script.
  You may need to adapt the `*.conf` files depending on your environment.

      cd deployable/roles/idp-test-client/files/
      PKI_PATH="<path to the directory containing the ca.crt file>" ./gen_certs.sh

- Jaeger:
  Certificates are located in `deployable/roles/jaeger-query/files/`.
  If you need to generate new certificates, you can use the `gen_certs.sh` script.
  You may need to adapt the `*.conf` files depending on your environment.

      cd deployable/roles/jaeger-query/files/
      PKI_PATH="<path to the directory containing the ca.crt file>" ./gen_certs.sh

- Keycloak:
  Certificates are located in `deployable/roles/keycloak/files/`.
  If you need to generate new certificates, you can use the `gen_certs.sh` script.
  You may need to adapt the `*.conf` files depending on your environment.

      cd deployable/roles/keycloak/files/
      PKI_PATH="<path to the directory containing the ca.crt file>" ./gen_certs.sh

- Kibana:
  Certificates are located in `deployable/roles/kibana/files/`.
  If you need to generate new certificates, you can use the `gen_certs.sh` script.
  You may need to adapt the `*.conf` files depending on your environment.

      cd deployable/roles/kibana/files/
      PKI_PATH="<path to the directory containing the ca.crt file>" ./gen_certs.sh

- Kubernetes:
  Certificates are located in `deployable/roles/kubernetes/files/certs/`.
  If you need to generate new certificates, you can use the `gen_certs.sh` script.
  You may need to adapt the `*.conf` files depending on your environment.

      cd deployable/roles/kubernetes/files/certs/
      PKI_PATH="<path to the directory containing the ca.crt file>" ./gen_certs.sh

  You can generate a new certificate for the service-account used by Kubernetes:

      cd deployable/roles/kubernetes/files/
      openssl genrsa -out service-account-key-file.key 4096

  You can generate a new uuid and place it in `deployable/roles/kubernetes/files/tokens`:

      uuidgen

  You must also specify the IP of the DNS of your infrastructure in `deployable/roles/kubernetes/vars/main.yml`:

      upstream_dns: ["192.168.1.1"]

- Sentry:
  Certificates are located in `deployable/roles/sentry/files/`.
  If you need to generate new certificates, you can use the `gen_certs.sh` script.
  You may need to adapt the `*.conf` files depending on your environment.

      cd deployable/roles/sentry/files/
      PKI_PATH="<path to the directory containing the ca.crt file>" ./gen_certs.sh

- Unbound:
  To correctly resolve domain names inside the cluster, we run an Unbound instance on every cluster node.
  Adapt the configuration in the file `deployable/roles/resolver/vars/main.yml` with the IP of your DNS server and your domain name:

      local_dns: 192.168.1.1
      domain: cloudtrust.io

Once you are done configuring, commit and push your modifications to the config repository.


### Deployment

In this part, we will deploy the components that compose the CloudTrust solution on the cluster VMs.

Infrastructure components:
- Kubernetes
- Ceph

Business components:
- PostgreSQL
- Elasticsearch
- Logstash
- InfluxDB
- Redis
- Kibana
- Jaeger
- Flaki
- Sentry
- Keycloak
- Grafana

The mandatory business components are PostgreSQL and Keycloak.

Clone the `on-premise-runtime` repository:

    git clone https://github.com/cloudtrust/on-premise-runtime.git
    cd on-premise-runtime

Install the required components in a python virtual environment called `venv`:

    dnf install libselinux-python python2-virtualenv
    virtualenv-2 --system-site-packages venv
    . venv/bin/activate
    pip install -r requirements.txt

Create a file `<env>-cluster-vars.yml` referencing the configuration repository used in the previous section.
It must contain the following entries:

    config_repo: <config-repo-url>
    # config_repo_name is a name used locally
    config_repo_name: dev
    config_git_tag: master

Configure the project.
This step initializes the `on-premise-runtime` project with the settings defined in the configuration repository.

    ansible-playbook config.yml --extra-vars "vars_file=<env>-cluster-vars.yml" -t deploy

Copy your ssh key to each node:

    ssh-copy-id root@<node-ip>

Run the resolver playbook.
This ensures that the network configuration will be correctly set on each node.

    ansible-playbook resolver.yml -i inventories/main.ini

Deploy Kubernetes:

    ansible-playbook kubernetes.yml -i inventories/main.ini

At this point, you can perform a few quick tests by logging on one of the node:

    [root@admin]# ssh root@dev-02

    # Every pod should be in the "Running" status
    [root@dev-02 ~]# kubectl get pods --all-namespaces
    NAMESPACE       NAME                                           READY     STATUS    RESTARTS   AGE
    default         kube-apiserver-dev-01.cloudtrust.io            1/1       Running   0          14m
    default         kube-apiserver-dev-02.cloudtrust.io            1/1       Running   0          14m
    default         kube-apiserver-dev-03.cloudtrust.io            1/1       Running   0          14m
    default         kube-controller-manager-dev-01.cloudtrust.io   1/1       Running   3          1d
    default         kube-controller-manager-dev-02.cloudtrust.io   1/1       Running   3          1d
    default         kube-controller-manager-dev-03.cloudtrust.io   1/1       Running   2          1d
    default         kube-scheduler-dev-01.cloudtrust.io            1/1       Running   5          1d
    default         kube-scheduler-dev-02.cloudtrust.io            1/1       Running   3          1d
    default         kube-scheduler-dev-03.cloudtrust.io            1/1       Running   2          1d
    ingress-nginx   default-http-backend-5c6d95c48-bl7jz           1/1       Running   2          1d
    ingress-nginx   nginx-ingress-controller-5w7jd                 1/1       Running   252        1d
    ingress-nginx   nginx-ingress-controller-gmb49                 1/1       Running   554        1d
    ingress-nginx   nginx-ingress-controller-jd59m                 1/1       Running   531        1d
    kube-addons     kube-dns-6467895fbd-xp44g                      3/3       Running   0          8m

    # The IP addresses for the endpoints should be the internal IPs of the nodes
    [root@dev-02 ~]# kubectl get endpoints kubernetes
    NAME         ENDPOINTS                                               AGE
    kubernetes   192.168.1.11:8443,192.168.1.12:8443,192.168.1.13:8443   1d

    # Verify that Flannel is binded to the correct network interface
    [root@dev-02 ~]# cat /etc/sysconfig/flanneld
    ...
    FLANNELD_IFACE="ens4"
    ...
    [root@dev-02 ~]# ifconfig
    ...
    ens4: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.1.12  netmask 255.255.255.0  broadcast 192.168.1.255
    ...


Before deploying Ceph, the docker image `cloudtrust-baseimage` must be built.
This may take a few minutes, due to running `dnf update && dnf install ...` in the image.
You will need to specify the path to the python virtual environment in the ansible variable `ansible_python_interpreter`, such as:

    ansible-playbook baseimage.yml -i inventories/local.ini -e "ansible_python_interpreter=/home/laurent/Desktop/CloudTrust/on-premise-runtime/venv/bin/python"

If everything succeeds, you should see a result like this when you list docker images:

    [root@admin]# docker images
    REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
    cloudtrust-baseimage   f27                 7f8cc043849e        37 seconds ago      528 MB
    docker.io/fedora       27                  9110ae7f579f        5 months ago        235 MB

Now that we have the baseimage, deploy Ceph.
It may take a few more minutes to complete:

    ansible-playbook ceph.yml -i inventories/main.ini

** TODO [LVA] ** Describe how to verify that Ceph is running

Deploy PostgreSQL:

    ansible-playbook postgresql.yml -i inventories/main.ini

The first time you deploy, the database is not initialized.
To initialize it, you must enter the container and perform some configuration changes:

    ssh root@<node-ip>
    kubectl exec -it postgresql-0 /bin/bash
    # Change the user and group
    chown postgres:postgres /var/lib/pgsql/data/
    initdb --pwfile /var/lib/pgsql/postgres.pwd /var/lib/pgsql/data/

Edit the file `/var/lib/pgsql/data/pg_hba.conf`, near the bottom of the file, search for the line:

    host    all             all             127.0.0.1/32               trust

Replace the line by:

    host    all             all             0.0.0.0/0               password

Edit the file `/var/lib/pgsql/data/postgresql.conf`, search for the line:

    #listen_addresses = 'localhost'

Replace the line by:

    listen_addresses = '*'

Exit the container and restart the pod:

    kubectl delete pod postgresql-0

Once the pod has restarted, you can check that postgres is running by connecting to the DB:

    # You can install psql by running `dnf install postresql`
    psql -h postgresql -U postgres

Deploy Elasticsearch:

    ansible-playbook elasticsearch.yml -i inventories/main.ini

Deploy Logstash:

    ansible-playbook logstash.yml -i inventories/main.ini

Deploy Influx:

    ansible-playbook influx.yml -i inventories/main.ini

Deploy Redis:

    ansible-playbook redis.yml -i inventories/main.ini

Deploy kibana:

    ansible-playbook kibana.yml -i inventories/main.ini

Deploy jaeger:

    ansible-playbook jaeger.yml -i inventories/main.ini

Deploy flaki:

    ansible-playbook flaki.yml -i inventories/main.ini

Deploy sentry:

    ansible-playbook sentry.yml -i inventories/main.ini

Deploy keycloak:

    ansible-playbook keycloak.yml -i inventories/main.ini

Deploy grafana:

    ansible-playbook grafana.yml -i inventories/main.ini

** TODO [LVA] ** Describe service tests


# Troubleshooting

- When building docker images, you may encounter an error with `rpmdb`, like this:

      error: rpmdb: BDB0689 Packages page 1399 is on free list with type 7
      error: rpmdb: BDB0061 PANIC: Invalid argument
      error: db5 error(-30973) from dbcursor->c_put: BDB0087 DB_RUNRECOVERY: Fatal error, run database recovery
      error: error(-30973) adding header #326 record

  Unfortunately, this error happens intermittently, so it has not been possible to fix it.
  The following workaround should allow you to build the image:

  1. Find the role corresponding to the image and comment the part where the Git repository is cloned.
    For example, in `roles/logstash/tasks/build.yml`:

         ---
         #- name: Pull repository
         #  git:
         #    repo: '{{ git_repo }}'
         #    dest: '{{ pull_path }}'
         #    version: '{{ git_tag }}'

         - name: Create build context
           file:
             path: '{{ context }}'
             state: directory
         ...

  2. Find the dockerfile and add `touch /var/lib/rpm/*` at the start of the line which runs `dnf`.
    For example, in `pull/logstash/dockerfiles/cloudtrust-logstash.dockerfile`:

         FROM cloudtrust-baseimage:f27
         ...
         RUN touch /var/lib/rpm/* && dnf update -y && \
         dnf install -y java java-1.8.0-openjdk.x86_64 logstash && \
         dnf clean all
         ...

  3. Re-execute the ansible playbook.
