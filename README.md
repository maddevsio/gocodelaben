# Making a simple database for geo-data

Hello, gopher. Well, if you are not a gopher and want to become one, hello too. I propose to combine two things in this codelab: to learn Go, as a programming language, and, maybe, to learn a couple of new things for yourself.

# The evironment

You will need the following:

1. Installed [Go Programming Language](https://golang.org)
2. Configured `GOPATH` :trollface: (For 1.8 not relevant)
1. You are familiar with basic things in Go. [“Go” tour](https://tour.golang.org/) can help you with this

# Purpose of laboratory work

This laboratory work has two purposes:

1. Get an experience in Go
2. Learn how does the key-value of the repository work (redis, memcached)
3. How some indexes work.

Eventually, the database will be able to do the following things:

* Quick search by the key;
* Search for places near you;
* HTTP interface to the database;
* LRU / expire mechanisms for data storage;

By Go you will get the following knowledge:

* How does concurrency work;
* Work with basic syntactic things;
* Test experience in go;
* Basic things with Makefile;

# Table of contents

Этот воркшоп разделен на несколько частей.

* [Step 0. Setting of the problem](step00/README.md)
* [Step 1. What you need to know about testing and writing tests in Go.](step01/README.md)
* [Step 2. Hello world](step02/README.md)
* [Step 3: Design the HTTP API](step03/README.md)
* [Step 4. Make the HTTP API](step04/README.md)
* [Step 5. Split main.go into several packages](step05/README.md)
* [Step 6. Makefile, configuration and flags](step06/README.md)
* [Step 7. Add a data warehouse and look for the nearest drivers in a naive way](step07/README.md)
* [Step 8. Writing the first benchmark: why do we need it](step08/README.md)
* [Step 9. What is R-tree and why is it more effective than naive implementation?](step09/README.md)
* [Step 10. Implement the LRU (Part 1)](step10/README.md)
* [Step 11. Implement the LRU (Part 2)](step11/README.md)
* [Step 12: Making the repository consistent. Introducing the LRU](step12/README.md)
* [Step 13: Implement the repository in the API](step13/README.md)
* [Step 14. You have completed the course. Congratulations](step14/README.md)

# Community and resources
There are several places where you can find information about Go:

- [golang.org](https://golang.org)
- [godoc.org](https://godoc.org) here you can find the documentation for any package
- [Go language blog](https://blog.golang.org)

One of the most remarkable qualities of Go is its community.
### Communities and channels in Telegram

1. [@bishkekgophers](https://telegram.me/bishkekgophers) - Bishkek Gophers
2. [@devkg](https://telegram.me/devkg) - Developers of Kyrgyzstan
3. [@maddevsio](https://telegram.me/maddevsio) - The channel of our company, where we share all kinds of interesting things. We often speak about Go

### Communities in Slack

1. [golang-ru.slack.com](golang-ru.slack.com) - The Russian-speaking community of gophers
2. [gophers.slack.com](gophers.slack.com) - The English-speaking community of gophers. Invitation to get here [https://invite.slack.golangbridge.org/](https://invite.slack.golangbridge.org/)


### Подкасты

1. [GolangShow](https://golangshow.com) - Russian-language podcast about Go-language
2. [Gotime](http://gotime.fm) - English-language podcast about Go-language

### Остальное
- [Go Форум](https://forum.golangbridge.org/)
- [@golang](https://twitter.com/golang) and [#golang](https://twitter.com/search?q=%23golang) on Twitter.
- [Go+ community](https://plus.google.com/u/1/communities/114112804251407510571) on Google Plus.

### Благодарности

1. Francesc Campoy for his workshop [Building Web Applications with Go](https://github.com/campoy/go-web-workshop/)
2. Ashley McNamara for the picture in the 10th step. You can see other works in [repo](https://github.com/ashleymcnamara/gophers)
3. [Elena Grahovac](https://twitter.com/webdeva) for the review and feedback
