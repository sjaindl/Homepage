---
title: 'Data Structures & Algorithms Collection Part 3: Stacks & Queues'
date: '13:38 11/06/2020'
taxonomy:
    category:
        - blog
    tag:
        - 'Datastructures & Algorithms'
theme: magnet-theme
header_image: '0'
feed:
    limit: 10
hero_classes: 'text-dark title-h1h2 overlay-light hero-large parallax'
hero_image: unsplash-luca-bravo.jpg
blog_url: /blog
show_sidebar: false
show_breadcrumbs: true
show_pagination: true
subtitle: 'finding beauty in structure'
header_id: blog
badge_class: badge-color-blog
---

This is the third post of my "Data Structures & Algorithms Collection" series. 
In this blog post I'll walk through a sample problem that deals with stacks and queues and explain how to solve it.

===

Queue via Stacks: Implement a QueueWithStacks class which implements a queue using 2 stacks.

The main difference between a queue and stack is in what order elements are removed.
Stack is a LIFO data structure, where the last element added is the first being removed again.
Vice versa, Queue is a FIFO data structure, where the first element added is the first being removed again.

How can we take advantage of that?
Well, we can always push elements on the first stack. When we want to pop an element we have to reverse the order. We can do that by popping elements from the first stack and pushing onto the second stack. The top element on the second stack will be the searched one.

The performance can be improved by a lazy approach. Push operations always push on the first stack. Pop operations are then doing all the work. If the second stack is not empty we can just return the element. Elsewise we pop all elements from stack one and push onto stack 2.

The amortized time for that is O(1).

Here's the source code:

```
class QueueWithStacks<T: Comparable> {
    private var firstStack = Stack<T>()
    private var secondStack = Stack<T>()
    
    open func enqueue(value: T) throws {
        firstStack.push(val: value)
    }
    
    open func dequeue() throws -> T {
        if firstStack.isEmpty() && secondStack.isEmpty() {
            throw NSError(domain: "Stack: Invalid call", code: 0, userInfo: nil)
        }
        
        moveElements(from: firstStack, to: secondStack)
        
        return try secondStack.pop()
    }
    
    open func isEmpty() -> Bool {
        return firstStack.isEmpty() && secondStack.isEmpty()
    }
    
    private func moveElements(from: Stack<T>, to: Stack<T>) {
        while !from.isEmpty(), let node = try? from.pop() {
            to.push(val: node)
        }
    }
}
```

You can also check it out in my repository:

[QueueWithStacks](https://github.com/sjaindl/DataStructuresAlgs/blob/master/Sources/DataStructuresAlgorithms/SpecificAlgorithms/Stack/QueueWithStacks.swift)
