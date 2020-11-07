---
title: 'Data Structures & Algorithms Collection Part 2: Linked Lists'
date: '19:04 11/05/2020'
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

This is the second post of my "Data Structures & Algorithms Collection" series. 
In this blog post I'll walk through a sample problem with linked lists and explain how to solve it.

===

Palindrome Checker: Implement a function to check if a linked list is a palindrome.

What is a palindrome?

It is simply a list that's read the same from front to back and back to front. For example:

1 -> 2 -> 3 -> 4 -> 3 -> 2 -> 1

It could have an even or odd count.

One approach is to just create a copy of the linked list in reversed order. Then we can iterate through both linked lists step by step and compare the elements. All elements need to be equal, else it's no palindrome. After reaching the midpoint the iteration can be aborted, as all elements are checked from either original or reversed list - it is a palindrome. This approach needs linear extra space for the reversed linked list.

Another approach is to move forward to the midpoint of the linked list (taking into account whether the list has an odd or even count) and pushing each element on a stack. Then continue moving forward, popping the respective element from the stack and comparing it to the current element. This works because stacks store their element in FIFO order. 
(Hint: if we know the linked list size we can just count until we reach the midpoint. If we don't know the size we can use a fast-slow-runner-technique. Moving with the slow pointer 1 node and with the fast runner 2 nodes forward. When the fast runner reaches the end, the slow runner will be at the midpoint.).

This can also be done in a more elegant, recursive way. 
Our base case is when we reach the midpoint. From there on we return the next node (this is node + 1, + 2,... from midpoint), which is compared to the node the function call returns to (this is node - 1, - 2,... from midpoint). Actually we need to return values: The next node to compare and a bool indicating whether the palindrome check was successful previously - this can be done in a tuple in Swift or a wrapper class.


Here's the source code:

```
func isPalindrome(linkedList: SingleLinkedList<T>) -> Bool {
        guard let head = linkedList.head, linkedList.count > 1 else {
            return true
        }
        
        return isPalindrome(node: head, index: 0, count: linkedList.count).isPalindrome
    }
    
    private func isPalindrome(node: SingleNode<T>?, index: Int, count: Int) -> ResultWrapper<T> {
        if index >= Int(trunc(Double(count) / 2)) {
            let node = count % 2 == 0 /* even */ ? node : /* odd */ node?.next
            return ResultWrapper(node: node, isPalindrome: true)
        }
        
        var resultSoFar = isPalindrome(node: node?.next, index: index + 1, count: count)
        let equal = resultSoFar.node?.val == node?.val
        resultSoFar.isPalindrome = resultSoFar.isPalindrome && equal
        resultSoFar.node = resultSoFar.node?.next
        
        return resultSoFar
    }
    
    private struct ResultWrapper<T: Comparable> {
        var node: SingleNode<T>?
        var isPalindrome: Bool
    }
```

You can also check it out in my repository:

[PalindromeChecker](https://github.com/sjaindl/DataStructuresAlgs/blob/master/Sources/DataStructuresAlgorithms/SpecificAlgorithms/LinkedList/LinkedListPalindrome.swift)
