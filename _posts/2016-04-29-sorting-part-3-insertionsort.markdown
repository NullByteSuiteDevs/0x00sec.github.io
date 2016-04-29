---
layout: post
title:  "Sorting (Part 3.0) Insertion Sort"
date:   2016-04-29 19:49:00 -0700
categories: algorithm
tags:
 - sorting
 - insertion sort
author: oaktree
---
Hello World! Today I'll be talking about Insertion Sort.
<!-- more -->
# What Is Insertion Sort?

Insertion Sort is a particular `O(n^2)`, `o(n)`, sorting algorithm that goes through an array and, if an element `A` is smaller than the element to its left, it shifts all the elements greater than `A` to the right one space to insert `A` into the proper space.

<img src="http://img.wonderhowto.com/img/82/84/63593367594595/0/sorting-part-3-0-insertion-sort.w654.jpg"/>

It's worse case time complexity, `O(n^2)`, happens when the inputted array is in reverse order, since all the elements will have to be shifted and read.

The best case, `o(n)`, takes place when the array to sort is already sorted.

# Let's Ruby It

Source Code:

{% highlight ruby linenos %}
#! /usr/bin/env ruby
def sorted?(arr)
    for i in 1...arr.length
        return false if arr[i] < arr[i-1]
    end
 
    return true
end
# ignore, just input
puts "give me a string"
str = gets.chomp.split('')
strlen = str.length
# stop ignoring
 
# PAY ATTENTION HERE
# if the element to the left bigger than the element we are looking at, str[i]
# we need to put str[i] where it belongs.
# and we do this by temporarily storing the value of str[i] and then shifting
# all the elements of the array/str that are bigger than str[i] to the right
# until we arrive at str[i]'s new, rightful place
while !sorted?(str)
 
    for i in 1...strlen #exclusive range
 
        if str[i-1] > str[i]
            hold = str[i] # store value to INSERT  
 
            pos = i
 
            while pos > 0 && str[pos - 1] > hold
                str[pos] = str[pos - 1] # shift up
                pos -= 1
 
                # stop if we have shifted all the elements that are BOTH greater than str[i] AND
                # to the left of str[i]
            end
           
            # put str[i] in its rightful spot, just left of the last element we shifted, or the original spot of
            # the last element we shifted
            str[pos] = hold
        end
 
    end
 
end
puts str.join('')
{% endhighlight %}

In Insertion Sort, the left part of the array is sorted an the right part is unsorted. Any standalone element is technically sorted. Thus, we treat `str[0]` as sorted and move on to `str[1]`, which is why we start our for loop on `line 18` at `i = 1`.

On `line 19`, we check to see if `str[i-1] > str[i]`, meaning that we want to see if the element to the left of element `i` is bigger. If it is, our array is not yet sorted.

So, we have to move the value of `str[i-1]` to the spot of `str[i]` and then see if `str[i-2]` is also bigger than our original `str[i]`. Until `str[i-n]` is less than `str[i]`, or we reach the left end of the array, we must keep shifting elements to the right.

The while loop accomplishes the shifting and will continue to do so until we have shifted the last element that is greater than hold, our temporary variable for the original `str[i]`.

That's all there is to it!

# C++ Implementation

Before we start, I just want you to know that all the `vector<char>` stuff is simply a C++ dynamic array.

{% highlight c++ linenos %}
#include <iostream>
#include <string>
#include <vector>
#include <cstdlib>
 
using namespace std;
bool sorted(vector<char>& vec) {
    for (int i = 1, n = vec.size(); i < n; i++) {
        if (vec[i] < vec[i-1]) return false;
    }
 
    return true;
}
void insertionsort(vector<char>& vec) {
    int n = vec.size();
    /*
    * The first element in the array is treated as sorted,
    * so we go on to the second one, vec[1], which is why
    * the for loop starts at i = 1
    */
    while (!sorted(vec)) {
        for (int i = 1; i < n; i++) {
            // if the element to the left bigger than the element we are looking at, vec[i]
            // we need to put vec[i] where it belongs.
            // and we do this by temporarily storing the value of vec[i] and then shifting
            // all the elements of the array/vec that are bigger than vec[i] to the right
            // until we arrive at vec[i]'s new, rightful place
            if (vec[i-1] > vec[i]) {
                int hold = vec[i]; // hold value
 
                int pos = i; // track positive to insert `hold`
               
                while (pos > 0 && vec[pos - 1] > hold) {
                   
                    vec[pos] = vec[pos - 1];
                    pos--;
 
                    // but, if our original vec[i] is bigger than
                    // the element we shifted in the last iteration,
                    // we can stop shifting
                }
                // now we can put the original vec[i] where it belongs by accessing
                // the pos variable which holds the position we want to INSERT the
                // original vec[i] into
                vec[pos] = hold;
            }
        }
    }
}
// BELOW IS JUST INPUT, FEEL FREE TO OVERLOOK
int main() {
    cout << "give me a string" << endl;
    string s; getline(cin, s);
 
    vector<char> vec(s.begin(), s.end());
 
    if (!vec.empty()) insertionsort(vec);
 
    string str(vec.begin(), vec.end());
 
    cout << str << endl;
    return 0;
}
{% endhighlight %}

Ignore `main()`, as that is just the gathering of input. Focus on `insertionsort(...)`, which takes an array (`vector`) as its only parameter.

We do pretty much the same thing here as we did in Ruby. All that really changes is the syntax and that I called our array `vec` instead of `str`.

So, look at the comments and read what I wrote for the Ruby implementation.

# Conclusion

That's all for Insertion Sort. I encourage you all to run the programs yourselves and type them out to increase your understanding.

Next time I will be talking about Selection Sort, but not before a little gift of Bogo Sort!

Keep on hacking,

oaktree