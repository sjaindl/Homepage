---
title: 'Data Structures & Algorithms Collection Part 1: Arrays'
date: '18:50 11/05/2020'
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

Welcome to my series of "Data Structures & Algorithms Collection". For each blog post I'll walk through a sample problem and explain how to solve it.
This blog deals with arrays.

===

Zero Matrix: Write an algorithm such that if an element in an MxN matrix is 0, its entire row and column are set to 0.

Suppose we have this matrix:

[[1, 2,  0],
 
 [4, 5,  6],
 
 [7, 8,  0],
 
 [9, 10, 11]]

It should be set to (nullifying all eligible rows and columns with 0 - this is row 1 & 3 + column 3):

[[0, 0,  0],

 [4, 5,  0],

 [0, 0,  0],
 
 [9, 10, 0]]

A first approach might be to set all columns and rows to zero on the fly, while iterating through all rows and columns.
This doesn't work because it sets cells to zero we haven't checked yet. Soon the whole matrix may consist of zeroes.

As a workaround we could define a second matrix and flagging rows & columns while iterating through the input matrix. This works, but needs O(M * N) extra space for the matrix. We can do better.

We could use the first row and first column as temporary data store for our zero flags. Then, we first need to check whether the first row or column contains zeroes and memorize the result as boolean value (as we are destroying these cells - but so we are able to restore them).

Then we iterate through the matrix starting from column and row indices 1. If a cell is zero, the corresponding index 0 row and columns are marked with 0, too.

With that information in row & column index 0, we can then nullify all other cells and, finally, nullify row and column 0, if there initially was some zero value.

This approach needs no extra space, so it is constant O(1) space and has the best conceivable runtime of O(M * N).

Here's the source code:

```
func zeroMatrix(matrix: inout [[Int]]) {
        guard !matrix.isEmpty, !matrix[0].isEmpty else {
            return
        }
        
        //We use the first row and column to zero the matrix in place
        
        //So, first remember whether we have to nullify first row and column
        let nullifyFirstRow = checkNullifyFirstRow(matrix: matrix)
        let nullifyFirstColumn = checkNullifyFirstColumn(matrix: matrix)
        
        //Check remaining cols/rows
        for row in 1 ..< matrix.count {
            for column in 1 ..< matrix[0].count {
                if matrix[row][column] == 0 {
                    matrix[0][column] = 0
                    matrix[row][0] = 0
                }
            }
        }
        
        //Nullify remaining columns
        for column in 1 ..< matrix[0].count {
            if matrix[0][column] == 0 {
                for row in 1 ..< matrix.count {
                    matrix[row][column] = 0
                }
            }
        }
        
        //Nullify remaining rows
        for row in 1 ..< matrix.count {
            if matrix[row][0] == 0 {
                for column in 1 ..< matrix[0].count {
                    matrix[row][column] = 0
                }
            }
        }
        
        if nullifyFirstRow {
            for column in 0 ..< matrix[0].count {
                matrix[0][column] = 0
            }
        }
        
        if nullifyFirstColumn {
            for row in 0 ..< matrix.count {
                matrix[row][0] = 0
            }
        }
    }
    
    private func checkNullifyFirstRow(matrix: [[Int]]) -> Bool {
        for col in 0 ..< matrix[0].count {
            if matrix[0][col] == 0 {
                return true
            }
        }
        
        return false
    }
    
    private func checkNullifyFirstColumn(matrix: [[Int]]) -> Bool {
        for row in 0 ..< matrix.count {
            if matrix[row][0] == 0 {
                return true
            }
        }
        
        return false
    }
```

You can also check it out in my repository:

[ZeroMatrix](https://github.com/sjaindl/DataStructuresAlgs/blob/master/Sources/DataStructuresAlgorithms/SpecificAlgorithms/Arrays/ZeroMatrix.swift)
