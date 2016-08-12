---
layout: post
title: "Insertion Sort, Modern C++ and Others "
modified:
categories: blog
excerpt:
tags: [programming,algorithms,c++,ruby]
comments: true
keywords: programming,algorithms,c++,ruby,haskell,insertion,sort
date: 2015-01-18T20:20:10+05:30
description: Implementing Insertion Sort in Modern c++, ruby and haskell
---

Third semester of my college started and since it was the starting of year 2015 and I hope to make blogging a regular habit not a resolution because resoultions have become overrated and they rarely work, let's if it works. Anyways, this semester I have one of the most important subjects **Design and Analysis of Algorithms** and I plan to write my findings/thoguhts about alogrithms (and their implementations in modern c++) and things I have learned from this course so as I have archived notes for future references.

# Insertion Sort

One of the simplest algorithms that's best for small sets of inputs and based on the intutive method of manual sorting of playing cards. OCW lectures call the current/main index as key and all operations with key as the base.Gist is to start with second element of the array and go till end and each stop insert the element in already sorted list,which can be slightly improved using binary search for finding the insertion in the already sorted array.

# Implementation

As with every algorithm you get a psuedocode as part of the tradtion.

## Psuedocode
{% highlight bash %}
For i = 1 to n
   set key = arr[i]
   while( j > 0 and arr[j] < key) #Swap pairwise till it's not in the right place
        swap(key,arr[j])
        decrement j
{% endhighlight %}


## Classic C++

As I was implementing my algorithm I noticed that it was not playing nice with negative numbers so I searched the web for implementations and most of them were same, and it looked this 
{% highlight cpp %}
 # include <iostream>
 # include <vector> 
 using namespace std; //Yes, I'm aware of namespace pollution
 int main()
 {
vector<int> arr = {1,3,0,-55,78};
int i, j, tmp;

      for (i = 1; i < arr.size(); i++) {

            j = i;

            while (j > 0 and (arr[j - 1] > arr[j])) {

                  tmp = arr[j];
                  arr[j] = arr[j - 1];
                  arr[j - 1] = tmp;
                  j--;
}
for(int i=0; i<arr.size();i++)
   cout<<i<<endl;
{% endhighlight %}

## Making it modern

It pisses me off when people use C like convention,styles and functions in c++ and call it "C with classes" but reality is different, C++ is not what it used to be, it is now modern and has better library support which makes it more readable and efficient. Now this algorithm can be made compact replacing the swapping operation with **swap()** function present in the <algorithm> header. Swapping will now be 
{% highlight cpp %}
 swap(arr[j],arr[j-1]);
{% endhighlight %}

Replacing the for loop for output with the **range based for loops** 
{% highlight cpp %}
 for (auto i : arr)
         cout<<i<<endl;
{% endhighlight %}

Now we need three headers **<iostream>, <vector>, <algorithm>**, to make our code shorter we can just include <bits/stdc++.h>, the daddy of header files, which will work fine on GCC based compilers. So now code our code becomes
{% highlight cpp %}
 # include <bits/stdc++.h>
using namespace std;
int main()
{
    vector<int> arr = {1,3,0,-55,78};
    int i, j;
    for (i = 1; i < arr.size(); i++)
    {
        j = i;

        while (j > 0 and (arr[j - 1] > arr[j]))
        {
            swap(arr[j],arr[j-1]);

            j--;

        }

    }
    for(auto i : arr) cout<<i<<endl;
}

{% endhighlight %}
 which is definitely shorter than the previous one.

## Leveraging the power of STL
 While I was searching for algorithms in modern C++ I found this great thread on [stackoverflow](http://stackoverflow.com/questions/24650626/how-to-implement-classic-sorting-algorithms-in-modern-c) which discussed many great ways to use inbuilt STL's and this [blog](http://www.bfilipek.com/2014/12/top-5-beautiful-c-std-algorithms.html#insert) which discussed the implementation in detail, here's the code 
{% highlight cpp %}
for (auto i = begin(arr); i != end(arr); ++i)
    std::rotate(std::upper_bound(begin(arr), i, *i), i, std::next(i));
{% endhighlight %}

Which fits the whole logic in just two lines. Pretty cool, huh ?
Since I am also using ruby so porting the same in ruby dialect won't be a big issue
{% highlight ruby %}
    arr = [1,-98,56,89,0]
    (0..arr.length-1).each do |x|
      j = x
         while(j > 0 and (arr[j-1] > arr[j]))
              arr[j-1],arr[j] = arr[j],arr[j-1]
              j = j.pred
      end
end

puts arr
{% endhighlight %}

That's all for now :)
