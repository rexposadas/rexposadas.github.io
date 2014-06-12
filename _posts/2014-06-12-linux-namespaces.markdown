---
layout: post                                                                                                                  
title:  "An exercise in Linux Namespaces"
date:   2014-06-12 18:41:55
categories: linux
---

This following post is an exercise in Linux namespaces.  The images below shows what we will build. This is an exercise derived from the following [tutorial](http://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/). 

All the commands in this post should be executed as `root`.

I wrote a [bash script](https://gist.github.com/rexposadas/6ac98e2f421e609ec842) that you can run to get the desired setup.  

<img src="/images/namespaces.jpg" alt="Drawing" style="width: 700px;height: 400px;"/>

The following setup has these properties:

1. The host should be able to ping ns1, but not ns2.
2. Ns1 should be able to ping the host and ns2.
3. Ns2 should be able to ping ns1, but not the host.


Create the namespaces.

    $ ip netns add ns1
    $ ip netns add ns2
 
check that the namespaces has been created.  After running the command you should 
see ns1 and ns2 in the list.

    $ ip netns list
 
Create the virtual ethernet pairs

    $ ip link add v1 type veth peer name v11
    $ ip link add v2 type veth peer name v22
 
Verify that the veth pairs were created
    
    $ ip link list
 
Move the virtual ethernet pairs around to match the image above.

    $ ip link set v11 netns ns1
    $ ip link set v2 netns ns1
    $ ip link set v22 netns ns2
 
Verify that we set the links correctly. 

    $ ip netns exec ns1 ip route list  # you should see v11 and v2
    $ ip netns exec ns2 ip route list  # you should see v22
 
Configure interfaces. 

    $ ifconfig v1 10.1.1.1/24 up
    $ ip netns exec ns1 ifconfig v11 10.1.1.2/24 up
    $ ip netns exec ns1 ifconfig v2 20.1.1.1/24 up    
    $ ip netns exec ns2 ifconfig v22 20.1.1.2/24 up


Taking a look at the image again and revisit it's expected behavior: 

<img src="/images/namespaces.jpg" alt="Drawing" style="width: 700px;height: 400px;"/>

## Testing our setup.

### The host should be able to ping ns1, but not n2.  In the host machine do the following:

    $ ping 10.1.1.2  // v11. succeeds.     
    $ ping 20.1.1.2  // v22 on ns2. fails. 
    $ ping 20.1.1.1  // v2. fails. the v2/v22 pair connects ns1 with n2.
    
    
### Ns1 should be able to ping the host and ns2. 
    
    $ ip netns exec ns1 ping 10.1.1.1 // host. succeeds.
    $ ip netns exec ns1 ping 20.1.1.2 // ns2. succeeds. 
        
### Ns2 should be able to ping ns1, but not the host. 
        
    $ ip netns exec ns2 ping 20.1.1.1 // ns1. succeeds. 
    $ ip netns exec ns2 ping 10.1.1.1 // host. fails. 
        
        


