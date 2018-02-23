# Repository structure

The Cloudtrust go git repositories follow the structure recommended by the [go best practices](https://peter.bourgon.org/go-best-practices-2016/#repository-structure). For a microservice, it will look like:

```
github.com/cloudtrust/<repo>/
.
├── .gitignore
├── .travis.yml             // Configuration of the travis CI
├── Gopkg.lock              // Dep lock
├── Gopkg.toml              // Dep manifest
├── LICENSE
├── README.md
├── doc.go                  // There must be a go file at repository root, otherwise go get complains.
├── script               
│   └── build.sh            // Build script
├── bin         
│   └── <name>              // Executable produced by build.sh 
├── cmd
│   ├── <name>.go           // main.go
│   └── <name>_test.go
├── conf                    // Configuration files
│   └── DEV
│       └── <name>.yml
└── pkg
    ├── <name>
    │   ├── flatbuffer      // Flatbuffers
    │   │   ├── fb
    │   │   │   └── ...
    │   │   └── <name>.fbs
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