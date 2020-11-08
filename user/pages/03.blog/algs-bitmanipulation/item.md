---
title: 'Data Structures & Algorithms Collection Part 5: Bit Manipulation'
date: '22:56 11/08/2020'
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

This is the fifth post of my "Data Structures & Algorithms Collection" series. 
In this blog post I'll walk through a sample bit manipulation problem.

===

Given a real number between 0 and 1 (e.g. 0.77) that is passed in as a double, output the binary representation. If the number cannot be printed accurately with 32 characters, an error should be thrown.

How are real binary numbers represented?
* 1 / 2^1 = 0.5
* 1 / 2^2 = 0.25
* 1 / 2^3 = 0.125
* ...

By knowing that, one way is to repeatedly shift the number left by 1 and check whether the number is > 1 - in this case the MSB (most significant bit) was set. We can append 1 to the output and subtract 1, else we just append 0 to the result.

Here's the source code:

```
class BinaryToStringConverter {

    private let errorString = "ERROR"
    
    func binaryRepresentation(of number: Double) -> String {
        var binary = "."
        var currentNumber = number
        
        if currentNumber <= 0 || currentNumber >= 1 {
            return errorString
        }
        
        while currentNumber > 0 {
            if binary.count >= 32 {
                return errorString
            }
            
            let digit = currentNumber * 2 //Same as shift left by 1
            binary.append(digit >= 1 ? "1" : "0")
            currentNumber = digit >= 1 ? digit - 1 : digit
        }
        
        return binary
    }
}
```

You can also check it out in my repository:

[Binary To String Converter](https://github.com/sjaindl/DataStructuresAlgs/blob/master/Sources/DataStructuresAlgorithms/SpecificAlgorithms/BitManipulation/BinaryToStringConverter.swift)
