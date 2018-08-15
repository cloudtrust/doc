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
 - component image: This image installs a component and its dependencies (for instance keycloak and its bridge and its modules), and usually comes with a plethora of arguments to shape it. Typical arguments are 
   - config_repository and git_tag : An application needs configuration, those are stored (for now) on a git server, those parameters specify the repository and the tag/commit hash of what to pull.
   - service_git_tag : The dockerfile should be explicit about what commit it is issued from! So instead of `ADD`ing the common configuration, we `git clone` it instead.
   - service_url : Sometimes a component depends on other components, and the dockerfile must fetch them. This is the url at which it will find the binaries to run.

## Systemd In Containers

Systemd is a widely used Linux init system. It expects to be running as PID 1 and have full control over the system. Thankfully, a lot of work has been put into it to make it runnable inside an unprivileged container. This however requires that :
 - /tmp is a tmpfs
 - /run is a tmpfs
 - /sys/fs/cgroup is mounted and is readable
Docker run commands need to have the following flags : `--tmpfs /tmp`, `--tmpfs /run` and `-v /sys/fs/cgroup:/sys/fs/cgroup:ro`.

Furthermore, since systemd attempts to read cgroups, selinux needs to let that happen. So the selinux boolean `container_manage_cgroup` needs to be set to true:

    setsebool container_manage_cgroup on
