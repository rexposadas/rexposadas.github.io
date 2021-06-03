---
layout: post
title:  "Go bug found"
date:   2014-05-22 18:41:55
categories: go
---

I found an interesting bug in Go1.2.  

1. [Golang group discussion](https://groups.google.com/forum/#!topic/golang-dev/1bV0GgIrX4A)
2. [Issue tracker](https://code.google.com/p/go/issues/detail?id=7623)


Here is the code in [playground](http://play.golang.org/p/Niss3Ed1kd)

{% highlight go %}
    package main

    type I struct {
        Name string
        Age  int
    }
    type S struct {
        One   I
        Two   I
        Three I
        Four  I
        Five  I
        Size  I
    }

    var A S

    func init() {
        A.One = I{Name: "tom", Age: 22}
        A.Two = I{Name: "tom", Age: 22}
        A.Three = I{Name: "tom", Age: 22}
        A.Four = I{Name: "tom", Age: 22}
        A.Five = I{Name: "tom", Age: 22}
    }

    func main() {
    }

{% endhighlight %}

Run this code and you\ll get an error. I came across this bug in Go1.2 when I was experimenting with the init()
function at work.  The work around I found was to split the function.



{% highlight go %}

    func init() {
        A.One = I{Name: "tom", Age: 22}
        A.Two = I{Name: "tom", Age: 22}
        A.Three = I{Name: "tom", Age: 22}
    }

    func init(){
        A.Four = I{Name: "tom", Age: 22}
        A.Five = I{Name: "tom", Age: 22}
        Size  I
    }

{% endhighlight %}


The code above compiles. This will be fixed in Go1.3 
