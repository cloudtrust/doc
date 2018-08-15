# CloudTrust

## To start using CloudTrust

Search our documentation on [cloudtrust.io](http://cloudtrust.io/doc/)

Read the [deployment guide](chapter-deploy/Deployment_Procedure_Cluster.md)

## To modify this documentation

Clone the repository:

    git clone -b gh-pages https://github.com/cloudtrust/doc.git

Install the gitbook plugin:

    npm install -g gitbook-cli
    gitbook install

Do your modifications.

Build the documentation:

    gitbook build

Copy the static site files into the current directory.

    cp -R _book/* .

Commit your changes and do a pull request.
