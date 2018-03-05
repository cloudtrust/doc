# Configurations

We manage our configurations in configuration repositories, which are simply a way to dump and version control our configs. In those repos, we find configurations for most components, as well as secrets, that are used at the time of the build.

There are two folders inside these repositories:
 - *deploy*: Is used during docker build to deploy configuration files inside the image. Its structure mimics the image to make what is deployed where as clear as possible .
 - *deployable*: Contains ansible variables for deploying the specific environment. Its structure mimics the ansible tree structure for similar reasons as above.

The root folder is for everything specific to that environment but that does not end up in the image or container, for instance credentials used by testing tools etc...
