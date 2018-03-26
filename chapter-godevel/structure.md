# Repository structure

The Cloudtrust go git repositories follow the structure recommended by the [golang-standards](https://github.com/golang-standards/project-layout). For one of our microservice, it will look like:

```bash
github.com/cloudtrust/<repo>/
.
├── .gitignore
├── .travis.yml             // Configuration of the travis CI
├── Gopkg.lock              // Dep lock
├── Gopkg.toml              // Dep manifest
├── LICENSE
├── README.md
├── doc.go                  // There must be a go file at repository root, otherwise go get complains.
├── api
│   └── *.fbs               // Flatbuffers schemas,...
├── bin
│   └── <name>              // Executable produced by build.sh
├── cmd
│   └── <name>.go           // main: the executable will be named <name>
├── configs
│   └── <name>.yml          // Configuration files
├── examples
│   └── ...                 // Examples
├── internal
│   └── <name>              // Private application and library code
│       └── ...
├── scripts
│   └── build.sh            // Build script
└── pkg
    ├── <name>
    │   ├── mock            // Auto generated mocks (see section test)
    │   │   └── ...
    │   └── ...
    ├── health              // Health checks
    │   ├── mock
    │   │   └── ...
    │   └── ...
    └── middleware          // Middlewares common between services
        ├── middleware.go
        ├── middleware_test.go
        └── mock
            └── ...
```