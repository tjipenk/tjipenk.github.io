---
title: "A De(Tour) of Go"
layout: post
date: 2021-10-21 17:00
image: /assets/a-detour-of-go/gologo.png
headerImage: false
tag:
- go
- programming
category: blog
author: karolfilipczuk
description: Learn Go with a slight detour in Getting Started section.
---
![Go Logo](../assets/a-detour-of-go/gologo.png)

# A (De)Tour of Go
New commitment for myself: learn new programming language. I’m already fluent in Bash and Python, but I’m missing from any compiled language. After doing some research, I have end up with Go.  

I read here and there that Go official documentation and tutorials are decent place to start and I admit, [Getting Started](https://golang.org/doc/#getting-started) section is a great starting point. What bothers me is the order of tutorials and documentation within the section.

The original order:
1. Installing Go
2. Tutorial: Getting started
3. Tutorial: Create a module
4. Tutorial: Developing a RESTful API with Go and Gin
5. Writing Web Applications
6. How to write Go code
7. A Tour of Go

## What’s wrong?
I tried to follow and finish all of the docs one by one. While first three are quite approachable without preparing , the fourth (Tutorial: Developing a RESTful API with Go and Gin) is starting to be too complex to went through it and be confident about what you have read. If I could finish Getting Started section once again, I’d definitely took a detour.  
Lucky for you, you can learn on my mistake and take a more approachable way to mastering basic of Go. 

## The (De)Tour of Go
1. [Installing Go](https://golang.org/doc/install)
2. [Tutorial: Get started with Go](https://golang.org/doc/tutorial/getting-started.html)
3. [How to Write Go Code](https://golang.org/doc/code.html)
4. [Tutorial: Create a Go module](https://golang.org/doc/tutorial/create-module.html)
5. [A Tour of Go](https://tour.golang.org/)
    1. [Go by Example: Interfaces](https://gobyexample.com/interfaces)
    2. [How to use interfaces in Go](https://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go)
    3. [Concurrency — An Introduction to Programming in Go](https://www.golang-book.com/books/intro/10)
6. [Tutorial: Developing a RESTful API with Go and Gin](https://golang.org/doc/tutorial/web-service-gin.html)
7. [Writing Web Applications](https://golang.org/doc/articles/wiki/)

There you go. Fixed order for Getting Started section.  

**Tl;dr**: I believe this order makes it easier to learn Go.  
Keep reading if you want to know more about each point in A De(Tour) of Go 

### Installing Go and Get started with Go  
First to docs are the same. You will install Go, initialize a Go module, write classic „Hello, World!” code and import an external package.
These give an overview of total Go’s basics.  

### How to Write Go Code
Originally third doc was Tutorial: Create a Go module. I was wondering if it should stay at third spot or not. I’d say it doesn’t do big difference if you want to take Tutorial: Create a Go module or How to Write Go Code as first. What makes huge difference is to definitely take How to Write Go Code as fourth the latest.  
How to Write Go Code is more theoretical than the previous docs. There are code snippets that you can run, but the crucial take away from the docs is knowledge about:
- Code organization
- First program
- Installing program
- Importing packages from your and external modules
- Testing  

I’d recommend go through the doc, run snippets and bookmark it. This will be the knowledge base you will go back during the next two docs.  

### Tutorial: Create a Go module
Finishing How to Write Go Code beforehand, makes Tutorial: Create a Go module more approachable and you will understand it more deeply. 
You will create module and call your code from another module. You will handle the first error, write a simple test and get glance of Go’s modules like *„math”* and *„time”*. At the end you will finish it with compiling and installing your program.
See now why it was good choice to firstly take How to Write Go Code? It gave you look and fell for current doc.  

### A Tour of Go
The Tour of Go is great, interactive tool to get hands-on experience. It includes three parts:
1. Basics
2. Methods and Interfaces
3. Concurrency 

Basics will introduce you to packages, variables and functions. You will get more and more familiar with Go’s syntax learning about flow control statements like for, if, switch, defer. It also gives and overview with examples of types: structs, slices and maps. You can learn it pretty straightforward without knowing much about it beforehand.  

Methods and Interfaces are more cumbersome. If you have previous experience with other languages and you are familiar in general with methods, pointers, interfaces then you should you through first part without much issues. The second part though is about specific Go’s interfaces Error, Stringer, Reader and Images. They are not explained in details. They might be hard to understand and difficult to accomplish (each interface has it own  exercise). Therefore, I’d recommend to go through  [Go by Example: Interfaces](https://gobyexample.com/interfaces)  and  [How to use interfaces in Go](https://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go). You don’t need to learn it, just reading it will give you more confidence and you will be able to finish Methods and Interfaces knowing what you are doing.  

Concurrency. This topic can be sometimes overwhelming. Usually, when I feel like subject doesn’t make sense I start with basics before getting into the actual subject. If it’s you I suggest starting with definition of concurrency - what is it, why would you use it, what are benefits. Then get look and fell about concurrency by reading following [article](https://www.golang-book.com/books/intro/10). I don’t expect you to understand everything, but it will give an idea of concurrency in Go. Afterwards you will be ready to tackle concurrency subject in A Tour of Go.
### Developing a RESTful API with Go and Gin
After completing other docs and tutorials, developing RESTful API tutorial is rather straightforward. You are already familiar with basic Go syntax and you can focus on functionality, Gin web framework, general standalone program structure.
### Writing Web Applications
It’s the best  tutorial in the list. It goes step by step with great details. From basics like defining structs, through running web server, handling errors and finish with using closure. I highly recommend to finish other task at the end of tutorial. 

## Wrapping up
Hopefully you are now even more eager to start learning Go. My goal is to show a more approachable way for studying which will not make you frustrated. I have abandoned various learning subjects in the past just because of a steep learning curve. Personally, I like to think that any subject can be made easy to learn.  
The default order of  Go’s [Getting Started](https://golang.org/doc/#getting-started) can be still good choice. I believe The (De)Tour of Go order can give you more insights and better background knowledge and, as a result, better understanding of the subject.  

Let me know which path have you chosen.  
Thanks!

