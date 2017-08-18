# Step 6. Putting it all together and a little bit about Makefile

It's time to configure our application. For configuration, we will use the flag package [flag](https://godoc.org/flag)
```Go
import "flag"

func main() {
    bindAddr := flag.String("bind_addr", ":8080", "Set bind address")
    flag.Parse()
    a := api.New(*bindAddr)
    log.Fatal(a.Start())
}
```
In GO there are several rules of good taste:

1. There should be tests in your package
2. Your libraries should not write logs
3. Error messages should be written in lowercase
4. Your code must be formatted

To keep the code always formatted, in addition to saving triggers, you can run `go fmt ./...` and it will format all files. For an example I will share with, you a simple `Makefile` 
```make
TARGET=codelab

all: fmt clean build

clean:
    rm -rf $(TARGET)

depends:
    go get -u -v

build:
    go build -v -o $(TARGET) main.go

fmt:
    go fmt ./...

test:
    go test -v ./...
run:
    go run main.go
```

## Congratulations!

You will format all the code by making “make” and rebuild the project, and make run will start the project for you. In the [next](../step07/README.md) step we will write tests for our API and learn about coverage

