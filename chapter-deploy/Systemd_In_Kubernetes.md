# Systemd Containers on Kubernetes.

## The Pause container is PID 1

This has already been described in another section.
If PID namespace sharing is enabled (it is by default), the pause process will get PID 1, and systemd will not start up. This is solved by giving the `--disable-docker-shared-pid` flag to the kubelet.

## Failed to change ownership of session keyring: Permission denied

This is an issue that plagued us for a long time so I'm going to go in depth into what it is and why it happened.

Containers are different from VMs in that they run on the same kernel, and that is a limitation of the design (if they didn't they would be VMs, there's no way around it). As such, certain constructs that are kernel wide will be shared across containers. For instance the whole `/sys` filesystem, kernel modules, and in particular: kernel keyrings.

Kernel keyrings are a place to store user session secrets, like passwords etc... One component that actively makes use of the kernel keyrings is systemd. However, systemd in a container and systemd outside a container are two wildly different beasts. Systemd in a container is typically limited, especially if it isn't a privileged container. As such, any attempt to modify the keyring will be met with a "permission denied" error. So systemd attempting to touch keyrings will fail everytime and break the container...
Well we know that's not true since we are able to run systemd in docker containers, so why can't we do the same with kubernetes containers? It's not like our docker containers were more privileged, in fact, they're even less privileged as they have seccomp rules active.

And that is the main/only difference! Depending on what mechanism blocks the `keyctl` call, selinux, capabilities, seccomp, the return code of the syscall will be different. With SELinux and Capabilities, it's a rightful `ENOPERM` which means that systemd is not privileged enough to use keyrings. Systemd expects to be PID 1 and therefore the most root a root process could be, and therefore fails. However, when it is blocked by seccomp, it returns `ENOSYS` which could mean that the current kernel has no support for kernel keyrings. Well systemd is happy to oblige and will keep secrets somewhere else, since keyrings aren't supported on this kernel.

Now the big issue with kubernetes at first glance is that seccomp doesn't appear to be supported ( https://github.com/kubernetes/kubernetes/issues/20870 ), but digging in the source code we quickly find the file `pkg/security/podsecuritypolicy/seccomp/strategy.go` which shows us that it is indeed supported. From there we can get `staging/src/k8s.io/api/core/v1/annotation_key_constants.go` which shows us that there is a tag `seccomp.security.alpha.kubernetes.io/pod` that can be applied on a pod to give it a seccomp profile, and finally we can get that `v1.SeccompPodAnnotationKey: "docker/default"` returns the default docker seccomp profile, which confines `keyctl` appropriately.


sources : 
 - https://github.com/systemd/systemd/issues/7655
 - https://github.com/systemd/systemd/issues/6281

