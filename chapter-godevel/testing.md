# Testing with GO

For the units tests, we generates mocks with the [gomock](https://github.com/golang/mock) project. There is two steps involved, first we generate mocks for the desired interfaces with mockgen, then we use those mocks in the tests.
Make sure you have the latest versions of [mockgen, gomock](https://github.com/golang/mock#installation).

## Generate mock
Let's take a real example, where we want to generate a mock for the interfaces Flaki and Module in the project [flaki-service](https://github.com/cloudtrust/flaki-service/blob/master/pkg/flaki/module.go).

We add the directive 
```//go:generate mockgen -destination=./mock/module.go -package=mock -mock_names=Module=Module,Flaki=Flaki github.com/cloudtrust/flaki-service/pkg/flaki Module,Flaki```
directly in the go file (here module.go) as a comment, just below the package name.

* `-destination=./mock/module.go`: the mocks will be generated in `./mock/module.go`.
* `-package=mock`: the mock will be in package `mock`.
* `-mock_names=Module=Module,Flaki=Flaki`: the name of the mocks will be Module for Module and Flaki for Flaki. We override the default names (MockModule, MockFlaki) to avoid stutter. If we don't we end up with names like mock.MockModuleMockRecorder.
* `github.com/cloudtrust/flaki-service/pkg/flaki`: package where the interfaces are.
* `Module,Flaki`: Name of the interfaces to mock.

To generate the mocks, use ```go generate ./...``` from the root of the repository. It will scan all files and find the directives to generates the mocks. Note that this directive does not create directory, you need to manually create the `mock` directories.

## Use mocks
In a unit test, we need to:
* create mock controller. If you use several mocks in a test, you need only one controller for all of them.
* create mock.
* set expected results: for ```mockFlaki.EXPECT().NextIDString().Return("1", nil).Times(1)
``` we assert that the method `NextIDString()` is called with 0 parameters, is called only one time (.Times(1)), and we set its returned values to ("1", nil). If we want to simulate an error we simply do .Return("", fmt.Errorf("my error")). There are several other options you can use, just look at the documentation. 

```golang
import (
  ...
  "github.com/cloudtrust/flaki-service/pkg/flaki/mock"
  "github.com/golang/mock/gomock"
)

func Test(t *testing.T) {
  var mockCtrl = gomock.NewController(t)
  defer mockCtrl.Finish()
  var mockFlaki = mock.NewFlaki(mockCtrl)

  var m = NewModule(mockFlaki)

  mockFlaki.EXPECT().NextIDString().Return("1", nil).Times(1)
  var id, err = m.NextID(context.Background())
  assert.Nil(t, err)
  assert.Equal(t, flakiID, id)
}
```