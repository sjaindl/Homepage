---
title: 'Data Structures & Algorithms Collection Part 7: Sorting & Searching'
date: '00:43 12/30/2020'
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

This is the seventh post of my "Data Structures & Algorithms Collection" series. 
In this blog post I'll walk through a sample problem that deals with sorting & searching and explain how to solve it.

===

Search in Rotated Array: Given a sorted array of n integers that has been rotated an unknown number of times, write code to find an element in the array. You may assume that the array was originally sorted in increasing order.

Example:
Input: find 5 in {15, 16, 19, 20, 25, 1, 3, 4, 5, 7, 10, 14}
Output: 8 (the index of 5 in the array) 

As first approach we might think of binary search. Binary search classically looks at the center element and checks if it matches. If so, the desired index is found. If the searched value is smaller than the center value, the left half (low - center index - 1 is searched), else the right half (center index + 1 - high). This leads to a O(log n) runtime.

However, binary search can't be used directly because the array is rotated (so the left and right halves could contain lower as well as higher values, each). A modified version of binary search can come to rescue. 

The modification is to determine first whether the left/right half of the array is rotated:
- if array[low] < array[center], then the left half is ordered
- if array[high] > array[center], then the right half is ordered
.. we can then check whether the value lies in between the indices of the ordered side to determine whether to search left or right.

A bit more tricky is the case when array[low] == array[center]. Then the ordered half could be on both sides. 
The only possible check left is whether the right side array[high] is different to array[center]. If so, just the right side can be searched. Else both sides must be searched.

This approach needs no extra space, and has a runtime of O(log N) - the same as classical binary search - if there no or not many duplicates. If there are many duplicates the runtime grows to O(N).

Here's the source code:

```
    func search(array: [T], searched: T) -> Int? {
        guard !array.isEmpty else {
            return nil
        }
        
        return search(array: array, searched: searched, low: 0, high: array.count - 1)
    }
    
    private func search(array: [T], searched: T, low: Int, high: Int) -> Int? {
        let center = low + (high - low) / 2
        if array[center] == searched {
            return center
        }
        
        if high <= low {
            return nil
        }
        
        //search for ordered half
        if array[low] < array[center] {
            //left half is ordered
            if array[low] <= searched && searched <= array[center] {
                //within left range
                return search(array: array, searched: searched, low: low, high: center - 1)
            } else {
                //not within left range - search right
                return search(array: array, searched: searched, low: center + 1, high: high)
            }
        } else if array[high] > array[center] {
            //right half is ordered
            if array[center] <= searched && searched <= array[high] {
                //within right range
                return search(array: array, searched: searched, low: center + 1, high: high)
            } else {
                //not within right range - search left
                return search(array: array, searched: searched, low: low, high: center - 1)
            }
        } else {
            //could be on both sides
            if array[high] != array[center] {
                return search(array: array, searched: searched, low: center + 1, high: high)
            } else {
                if let result = search(array: array, searched: searched, low: center + 1, high: high) {
                    return result
                }
                return search(array: array, searched: searched, low: low, high: center - 1)
            }
        }
    }
```

You can also check it out in my repository:

[RotatedArraySearch](https://github.com/sjaindl/DataStructuresAlgs/blob/master/Sources/DataStructuresAlgorithms/SpecificAlgorithms/SortingAndSearching/RotatedArraySearch.swift)
