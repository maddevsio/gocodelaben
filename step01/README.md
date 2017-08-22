# Step 1. What you need to know about testing and writing tests in Go.

Testing is necessary for weaklings who can not write working code at the first attempt. :trollface: Therefore, there are many tools for writing tests in the world. Go is not an exception. Go has a `testing` package, whose functionality is enough for everyone.

The test files in GO are usually in the same folder as regular files. They can be recognized by the presence of `_test.go` in the file name. The compiler will understand that the `_test.go` files do not need to be included in the build, but when you run the `go test` tool, these files will be used just like that

For example, let's see how the `math` Go package is tested.

```Go
package math

import "testing"

func TestAverage(t *testing.T) {
  var v float64
  v = Average([]float64{1,2})
  if v != 1.5 {
    t.Error("Expected 1.5, got ", v)
  }
}
```

Launching the tests are going to the command `Gogo test`

```
$ cd /usr/local/go/src/math
$ go test
PASS
ok      math    0.010s
```

If you want.  The Go community also promotes to use the so-called table tests instead of copy-paste in the tests. In this case, we have pairs of initial value and result. And we run the tests in a loop

```Go
package math

import "testing"

type testpair struct {
  values []float64
  average float64
}

var tests = []testpair{
  { []float64{1,2}, 1.5 },
  { []float64{1,1,1,1,1,1}, 1 },
  { []float64{-1,1}, 0 },
}

func TestAverage(t *testing.T) {
  for _, pair := range tests {
    v := Average(pair.values)
    if v != pair.average {
      t.Error(
        "For", pair.values,
        "expected", pair.average,
        "got", v,
      )
    }
  }
```
[Documentation](http://godoc.org/testing) for the `testing` package
```Go
And if you tired of writing constantly

if smth != anoher {
   t.Error("Error")
}
```

That is, the [testify/assert](https://godoc.org/github.com/stretchr/testify/assert)
 package

## Congratulations!
You now know how to test in Go and by which tools. We will write tests in our project. There is no sense without them. Continuation in the [next](../step02/README.md) part

