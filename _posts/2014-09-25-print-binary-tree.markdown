---
layout: post                                                                                                                  
title:  "Printing a binary tree in Go"
date:   2014-09-25
categories: go
---

The **goal** is to print a binary tree in such a way that each level of the tree is printed in a new line. Also, each level of the tree should be printed from left to right.

This example is pretty cool since it'll demostrate how we can use Go channels to indicate the different levels of the tree.

### Reference

* Here is the [`entire code`](https://github.com/rexposadas/notes/blob/master/blog/trees/main.go)


### Let's write some code

Let's start with a tree implementation. 

Our tree definition simply looks like this: 

    type Node struct {
        Value int
        Left  *Node
        Right *Node
    }

A tree is made up of one or more nodes, hence the `Node` struct.

Let's make the print function.  The print function will print the tree taking into account the different levels of the tree. 

	// Print prints the entire tree from this node.	
	func (n *Node) Print() {

		if n == nil {
			return
		}
	
		// nodes in this channel are printed rightaway
		currentLevel := make(chan *Node, 1000)
	
		// notes in this channel will be printed after
		// a new line is generated.
		nextLevel := make(chan *Node, 1000)
	
		// Let's ready the current node for printing
		currentLevel <- n // root of the tree
	
		for len(currentLevel) > 0 {
	
			n := <-currentLevel
			fmt.Print(n.Value)
	
			if n.Left != nil {
				nextLevel <- n.Left
			}
			if n.Right != nil {
				nextLevel <- n.Right
			}
	
			if len(currentLevel) == 0 {
				fmt.Println("")
				swap(currentLevel, nextLevel)
			}
		}
	}


What gives us the ability to determine when the next level of the tree has been reached? It's the two channels `currentLevel` and `nextLevel`.  The `currentLevel` channel represents what to print now. The `nextLevel` channel represents node(s) which should be printed **after** we have inserted a new line. 


### Sample Tree

<img src="/images/tree.png" alt="Drawing"/>


### Let's print stuff

Let's create a main function which builds the tree above and prints it. 

	func main() {
	
		// top node, considered  the first level
		root := &Node{
			Value: 1,
		}
	
		// level 2
		n2 := &Node{Value: 2}
		n3 := &Node{Value: 3}
		root.Left = n2
		root.Right = n3
	
		// level 3
		n4 := &Node{Value: 4}
		n5 := &Node{Value: 5}
		n2.Left = n4
		n2.Right = n5
	
		n6 := &Node{Value: 6}
		n3.Right = n6
	
		fmt.Println("Printing root node:")
		root.Print()
	
		fmt.Println("Printing node n2:")
		n2.Print()
	
		fmt.Println("Printing node n3:")
		n3.Print()
	}

The output looks like this:

	Printing root node:
	1
	23
	456
	Printing node n2:
	2
	45
	Printing node n3:
	3
	6

Note that each level of the tree is represented on it's separate row.  Each level is printed from left to right according to the sample tree. 
