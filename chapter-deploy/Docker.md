# Intro to Linux containers

Linux containers are sets of processes isolated from the rest of the ecosystem. In essence, containers abstract away the kernel like VMs do with the hardware. This is possible via a number of kernel mechanisms, but most importantly, namespaces and cgroups.

Containers are made of 3 components: Namespaces, Cgroups and an Overlay filesystem. I am not going to delve into them, as there is a lot of great documentation already out there. Instead I am going to focus on Docker specifics, since that's what we use for Cloudtrust, and especially how we use it.

For a good overview on what makes containers possible on Linux, I recommend these articles : 
 - https://lwn.net/Articles/531114/
 - https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt
 - https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt
 - https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt
 - https://www.slideshare.net/jpetazzo/anatomy-of-a-container-namespaces-cgroups-some-filesystem-magic-linuxcon

For a good intro to docker :
 - https://nschoe.com/articles/2016-05-26-Docker-Taming-the-Beast-Part-1.html


## Docker in Cloudtrust

In Cloudtrust, our containers don't follow the guidelines by the docker team for practical reasons. We run multiple processes per container, which leads to having to run an init system as PID 1 in our containers. This helps out debugging and initial testing as we can edit configuration files at will and restart services inside the container with updated configuration, without having to rebuild/redeploy everything.
Furthermore, an init system is a great asset when trying to initialize containers (like create users, etc...), and is outright indispensable when a single service is made of multiple processes ( keycloak + keycloak_bridge ).

Our containers all follow the same schema for buildup:
 - base image: This image contains the git keys to pull from repos, as well as some system utilities. It installs git, systemd and the likes.
 - component image: This image installs a component and its dependencies (for instance keycloak and its bridge and its modules)
 - customer image: This image applies configuration to the component and the container.

Our different containers :
 - Postgresql: The Postgresql component image is very simple. We install postgresql, enable it, and set `/var/lib/pgsql` as a volume. The customer image is slightly more complex. It expects to have two files: `postgresql_init_sentry.service` and `postgresql_init_keycloak.service`, which are systemd units that run once postgresql is up, initialize users and databases required by sentry and keycloak, and then disable themselves. This mechanism is our main system for initializing containers at this moment.

 - Sentry: The sentry component image installs and enables sentry. The customer image is in charge of initializing the database and importing the saved configuration.

 - Keycloak: The component image installs keycloak and deploys modules where they should be. The customer image sets the standalone.xml, the user.json and the keycloak-bridge configuration/service.

 - Influx: The influx component image installs and enables influx it also installs the influx tools in their python virtual environment, and the init service file. The customer image deploys the config file used by the systemd service.

 - Grafana: The simplest of our images. it installs and enables grafana. The customer image is empty.


## Systemd In Containers

Systemd is the standard Linux init system. It expects to be running as PID 1 and have full control over the system. Thankfully, a lot of work has been put into it to make it runnable inside an unprivileged container. This however requires that :
 - /tmp is a tmpfs
 - /run is a tmpfs
 - /sys/fs/cgroup is mounted and is readable
Docker run commands need to have the following flags : `--tmpfs /tmp`, `--tmpfs /run` and `-v /sys/fs/cgroup:/sys/fs/cgroup:ro`.

Furthermore, since systemd attempts to read cgroups, selinux needs to let that happen. So the selinux boolean `container_manage_cgroups` needs to be set to true
