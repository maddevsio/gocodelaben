## Step 2. Hello world

Let's write something working. For example, a simple “Hello world” application. Open `main.go` in the editor and write the code.

```Go
package main

import "fmt"

func main() {
  fmt.Println("Hello world")
}
```

Now the project needs to be assembled and launched. There are several options for starting a project.
```
$ go run main.go
Hello world
```

```
$ go build -o helloworld main.go
$ ./helloworld
Hello world
```

Considering that we are writing a web application, let's make “hello world” on the web.
```Go
package main

import (
        "fmt"
        "log"
        "net/http"
)

func hello(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "<h1>Hello world</h1>")
}
func main() {
        http.HandleFunc("/", hello)
        log.Fatal(http.ListenAndServe(":9911", nil))
}
```
In this application we made a simple web server that will listen to the 9911 port at start and will return us the “`Hello world`” on every URL

Run and test it for yourself

## Congratulations!

You got something working. Continuation in the [next](../step03/README.md) part



