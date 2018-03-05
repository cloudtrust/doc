# Ansible in Cloudtrust

## Introduction to ansible

We use ansible to almost an embarassing degree in order to make our deployments as fast, reliable, and useable as possible. To this end, it's important to be familiar with this tool.

Ansible is simply a python wrapper over an ssh connection that will run *tasks* on an *inventory* of hosts. Tasks are typically *modules*, with parameters shaping what the module does. For instance the "copy" module can copy a file/dir from the ansible host to the ansible target.
Tasks are organized within *roles*, and are spread among multiple files to make things cleaner.
Roles are run via *playbooks* which are the only thing (as far as cloudtrust is concerned) that ansible should interact with.

## Ansible in cloudtrust

For every component that's deployable there is a playbook. We try to unify the tags that are used in each playbook as *build*, *deploy*, and *cleanup*. 

We will go over what the standard playbook do, and for that we will look at the keycloak playbook. The keycloak playbook simply runs the keycloak role, which runs tasks based on the given tags.
  - build: Depending on the value of multiple vars, ansible will clone repos locally and run `docker build` on the keycloak dockerfile with appropriate arguments. Once the build is done, the image is saved into a tar archive.
  - deploy: Once the build step is done, deploy will push the tar to the target (every host in the inventory). It will also render and deploy keycloak templates for kubernetes resources. Namely a service, a deployment, an ingress, and a tls certificate.
  - cleanup: Currently, cleanup only removes the locally cloned repository and the remote copy location. It has no effect on either the docker image or stuff instanciated in kubernetes.

### Ansible variables

Builds and deploys by their nature cannot be the same everytime everywhere, multiple environment will typically vary. For example a dev will have self-signed certificate but production will need an actual certificate. Passwords and other secrets will also vary from place to place, and we might like having the option to deploy something other than the tip of the dev branch.
In short, we need a way to customize deployments according to environments, and this is where variables come in.

There are variables defined at 4 different places, with different priorities, and different expected use.
 - group_vars: These are the most global variables, they will be available from every role, and are typically for general "state of the cluster" type things, such as IP ranges, cluster name, etc...
 - role defaults: In the *defaults* folder of most roles, there are variables specific to that role set to appropriate settings. These typically include names, paths... These should not change that often as they don't relate to things that useful.
 - role vars: This is in the vars folder of some roles. This contains variables that are specific to that role and impact functional things. These typically include URLs for repositories, and commit hashes/versions. These win over role defaults if a variable is specified multiple times.
 - extra vars: These add variables at runtime and are added to a playbook execution using `-e`. These are extremely specific use-cases that are added on a case-per-case basis, such as *force* which will force a docker image to be rebuilt without using cache, or *docker_http_proxy* which will add a proxy inside the docker service file.


To set environment specific variables, we have environment repositories, which contain the variables required for proper initialization. One such is the dev-config repository, which contains the dev configuration.
Along with role variables, these repositories can come with files (certificates, tokens...) that need to be deployed

Preparing an environment for deployment is done via the `config.yml` playbook, which simply clones the repository (url and tag passed by extra var) and syncs its content, mainly role variables and role files.




