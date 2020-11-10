---
title: 'Data Structures & Algorithms Collection Part 6: Recursion'
date: '22:46 11/10/2020'
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

This is the 6. post of my "Data Structures & Algorithms Collection" blog series. This blog deals with a recursion problem using dynamic programming.

===

Stack of Boxes: You have a stock of n boxes, with widths wi, heights hi and depths di. The boxes cannot be rotated and can only be stacked on top of each other if each box in the stack is strictly larger than the box above it in width, height and depth. Implement a method to compute the height of the tallest possible stack. The height of a stack is the sum of the heights of each box.

The bruteforce solution to this problem obviously would be to just try all valid combinations of all boxes. This would result in a very bad runtime - we can do better.

The problem states that each box must be larger than the previous one in terms of width, height and depth. We can use that property and sort by any property - e.g. by height first in O(n log n) time. We know that the final result must be in the resulting order (probably not including all boxes). We effectively eliminated the height dimension with this initial sort.

Now we need to find the largest stack by trying all stack combinations that can be formed by this ordering. We can start including the first box, then starting from the second box, and so on. Of course we need to check for each step if it is a valid combination also using width and depth. We don't need to check the height, as we already have sorted by height. This results in still at least O(n * n!) runtime, as there are n! permutations with a recurive call stack of depth n each.

We can still optimize it further by using dynamic programming. We can declare a hashmap which basically caches the largest result found at each stack index. We pass that hashmap down the recursive call stack and can use it for valid indices. Alternatively, if the box is represented with a class, a maxHeight property can be added to the class.

Here's the source code:

```
class StackOfBoxes {
    
    func maxHeight(of stack: [Box]) -> Int {
        if stack.isEmpty {
            return 0
        }
        
        let sorted = stack.sorted(by: >)
        var maxHeight = 0
        
        for (index, box) in sorted.enumerated() {
            if box.height > maxHeight {
                maxHeight = box.height
            }
            
            for previous in 0 ..< index {
                if isValid(first: box, second: sorted[previous]) && sorted[previous].maxHeight + box.height > box.maxHeight {
                    box.maxHeight = box.height + sorted[previous].maxHeight
                    
                    maxHeight = max(maxHeight, box.maxHeight)
                }
            }
        }
        
        return maxHeight
    }
    
    private func isValid(first: Box, second: Box) -> Bool {
        return first.width < second.width && first.height < second.height && first.depth < second.depth
    }
}

class Box: Equatable, Comparable {
    public let width: Int
    public let height: Int
    public let depth: Int
    
    public var maxHeight: Int
    
    public init(width: Int, height: Int, depth: Int) {
        self.width = width
        self.height = height
        self.depth = depth
        
        maxHeight = height //max height together with other boxes is at least == height of the box itself
    }
    
    public static func < (lhs: Box, rhs: Box) -> Bool {
        return lhs.height < rhs.height
    }
    
    public static func == (lhs: Box, rhs: Box) -> Bool {
        return lhs.width == rhs.width && lhs.height == rhs.height && lhs.depth == rhs.depth
    }
}
```

You can also check it out in my repository:

[StackOfBoxes](https://github.com/sjaindl/DataStructuresAlgs/blob/master/Sources/DataStructuresAlgorithms/SpecificAlgorithms/RecursionDynamicProgramming/StackOfBoxes.swift)
