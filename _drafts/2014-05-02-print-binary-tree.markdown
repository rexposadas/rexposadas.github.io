---
layout: post                                                                                                                  
title:  "Printing a binary tree in Go"
date:   2014-05-22 18:41:55
categories: go
---

# Printing a binary tree in Go.  

This is a little tricky since trees obviously has layers. For example:


Print a binary.  Each row of the tree should be printed on a separate line.
package main

Let's start with a tree implementation. 



    type Node struct {
        Value int
        Left  *Node
        Right *Node
    }
    func (n *Node) addLeft(l *Node) {
        n.Left = l
    }
    func (n *Node) addRight(r *Node) {
        n.Right = r
    }

Most languages would implement a tree like the above. But since this is Go, we can get rid of the "setter and getters".  Our tree definition simply looks like this: 

    type Node struct {
        Value int
        Left  *Node
        Right *Node
    }

Let's make the print function.  The print function will print the tree taking into account the different levels of the tree. 

    func print(n *Node) {

        if n == nil {
            return
        }
        
        q1 <- n // root of the tree

        for len(q1) > 0 {

        nn := <-q1
        fmt.Print(nn.Value)

        if nn.Left != nil {
            q2 <- nn.Left
        }
        if nn.Right != nil {
            q2 <- nn.Right
        }

        if len(q1) == 0 {
            fmt.Println("")
            swap(q1, q2)
        }
    }
}

For this exercise we know that q1 is empty.

    func swap(q1, q2 chan *Node) {

        for len(q2) > 0 {
            n := <-q2
            q1 <- n
        }

        q2 = make(chan *Node)
        return
    }


Main: 

    func main() {

    root := &Node{
        Value: 1,
    }

    // row 2
    n2 := &Node{Value: 2}
    n3 := &Node{Value: 3}
    root.addLeft(n2)
    root.addRight(n3)

    // row 3
    n4 := &Node{Value: 4}
    n2.addLeft(n4)

    n5 := &Node{Value: 5}
    n3.addRight(n5)

    print(root)
    fmt.Println("\n\n")
    print(n3)
}
