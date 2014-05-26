<!---
---
layout: post
title:  "Go bug found"
date:   2014-05-22 18:41:55
categories: go
---
-->

I found an interesting bug in Go 1.2.  You can find details here: 
[Golang group discussion](https://groups.google.com/forum/#!topic/golang-dev/1bV0GgIrX4A)
[Issue tracker](https://code.google.com/p/go/issues/detail?id=7623)

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
