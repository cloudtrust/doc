# Guidelines

## Recommended reading

* An introduction de Go: <https://talks.godoc.org/github.com/davecheney/introduction-to-go/introduction-to-go.slide#1>
* Go reference: <https://golang.org/doc/effective_go.html>
* 50 Shades of Go (Traps, Gotchas, and Common Mistakes for New Golang Devs): <http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/>
* Go design and philosophy: <https://talks.golang.org/2012/splash.article>

## IDE

We recommand VS Code (<https://code.visualstudio.com/docs/setup/linux>), but you are free to use any other IDE. You should just configure it to run the following tools on save:

* gofmt
* golint
* govet

## Cloudtrust naming conventions

* Logs: xxx\_xxx (lowercase, separated with "\_")
* Flags: xxx-xxx (lowercase, separated with "-")
* Config: xxx-xxx (lowercase, separated with "-")
* Packages import: xxx\_xxx (lowercase, separated with "\_")
* Variables: See Go naming conventions below.

In Cloudtrust, we use structured logging with the go-kit logger. All messages are key/value oriented.
The keys must contain lower case words separated with "_" and should NOT CONTAIN SPACES, otherwise the logger does not log anything.
See the section on Logging for the standardised keys used on the Cloudtrust project.

## Go naming

You should follow [those](https://talks.golang.org/2014/names.slide) naming conventions.
