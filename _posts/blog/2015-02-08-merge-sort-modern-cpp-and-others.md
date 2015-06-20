---
layout: post
title: "Merge Sort, Modern C++ and Others "
modified:
categories: blog/
excerpt: "Merge Sort and Modern C++"
tags: [programming,algorithms,c++,ruby]
comments: true
description: Implementing Insertion Sort in Modern c++, ruby and haskell
date: 2015-02-08T19:32:41+05:30
---

So this is my second post in implementing and learning (hopefully) some of the fundamentals algorithms
of Computer Science. This time it's <b>Merge Sort </b>, which uses the <b>Divide and Conquer </b> approach to give the result within
<b> O (n log(n)) </b>. MIT's OCW has a great series of lecture on the same and I loved the teaching style of Prof. Charles Leiserson, he simplifies most of the stuff plus he's the <b> L </b>in the famous <b>CLRS</b>.
Well approach of merge sort is simple and quite powerful and lecture made it look easy, but I have this weak point when it comes to implementation of some powerful and recursive algorithms (gotta improve that).

# Implementation

As I said I am not good at implementing algorithms, so I searched the web for implementation and found a great one at
[BogotoBogo](http://www.bogotobogo.com/Algorithms/mergesort.php), anyways gist of the algorithm can be laid out concisely
in the pseudocode

## PseudoCode
 This is for the Top Down implementation taken from  [Wikipedia](http://en.wikipedia.org/wiki/Merge_sort#Top-down_implementation_using_lists)
 Divide part
{% highlight bash %}
function merge_sort(list m)
    // Base case. A list of zero or one elements is sorted, by definition.
    if length(m) <= 1
        return m

    // Recursive case. First, *divide* the list into equal-sized sublists.
    var list left, right
    var integer middle = length(m) / 2
    for each x in m before middle
         add x to left
    for each x in m after or equal middle
         add x to right

    // Recursively sort both sublists.
    left = merge_sort(left)
    right = merge_sort(right)
    // *Conquer*: merge the now-sorted sublists.
    return merge(left, right)
{% endhighlight %}

Conquer and Merge Part

{% highlight bash %}
 function merge(left, right)
    var list result
    while notempty(left) and notempty(right)
        if first(left) <= first(right)
            append first(left) to result
            left = rest(left)
        else
            append first(right) to result
            right = rest(right)
    // either left or right may have elements left
    while notempty(left)
        append first(left) to result
        left = rest(left)
    while notempty(right)
        append first(right) to result
        right = rest(right)
    return result

{% endhighlight %}


##In simple words
 {% highlight bash %}
 Divide the list into halves : left and right
   Divide each half recursively into two halves until there is only one element
 Then call merge
   Store the sorted list into another list
     Sorting happens in a simple way
        Take the head of both list and store the smaller one before the larger and delete them from the old list
        If there is one list, simply copy the contents
  {% endhighlight %}
  
## C++
{% highlight cpp %}
#include <iostream>
#include <vector>

using namespace std;

void print(vector<int> v)
{
	for(int i = 0; i < v.size(); i++) cout << v[i] << " ";
	cout << endl;
}

vector<int> merge(vector<int> left, vector<int> right)
{
   vector<int> result;
   while ((int)left.size() > 0 || (int)right.size() > 0) {
      if ((int)left.size() > 0 && (int)right.size() > 0) {
         if ((int)left.front() <= (int)right.front()) {
            result.push_back((int)left.front());
            left.erase(left.begin());
         } 
	 else {
            result.push_back((int)right.front());
            right.erase(right.begin());
         }
      }  else if ((int)left.size() > 0) {
            for (int i = 0; i < (int)left.size(); i++)
               result.push_back(left[i]);
            break;
      }  else if ((int)right.size() > 0) {
            for (int i = 0; i < (int)right.size(); i++)
               result.push_back(right[i]);
            break;
      }
   }
   return result;
}

vector<int> mergeSort(vector<int> m)
{
   if (m.size() <= 1)
      return m;
 
   vector<int> left, right, result;
   int middle = ((int)m.size()+ 1) / 2;
 
   for (int i = 0; i < middle; i++) {
      left.push_back(m[i]);
   }

   for (int i = middle; i < (int)m.size(); i++) {
      right.push_back(m[i]);
   }
 
   left = mergeSort(left);
   right = mergeSort(right);
   result = merge(left, right);
 
   return result;
}

int main()
{
   vector<int> v;

   v.push_back(38);
   v.push_back(27);
   v.push_back(43);
   v.push_back(3);
   v.push_back(9);
   v.push_back(82);
   v.push_back(10);

   print(v);
   cout << "------------------" << endl;

   v = mergeSort(v);

   print(v);
}
{% endhighlight cpp %}

This code works flawlessly but it can be shortened and improved, like removing all the c-style casting <b> (int) </b>, since
removing doesn't breaks the code. Replacing <code> ||, && </code> with actual words like <code> or, and </code>.
Replacing the print fucntion by a lambda. So instead of writing print like
{% highlight cpp %}
void print(vector<int> v)
{
	for(int i = 0; i < v.size(); i++) cout << v[i] << " ";
	cout << endl;
}
{% endhighlight %}
we can write it like this, using the range based for loops
{% highlight cpp %}
 void print(vector<int> v)
   {
   for(auto i : v) cout << v; //Range based for loops
   }
{% endhighlight %}
or using the awesome lambdas
{% highlight cpp %}
void print(vector<int> v)
{
 auto print = [](int i){cout <<i<<" ";};
 for_each(begin(v),end(v),print);
}
{% endhighlight %}
Instead of having an extra vector to hold the merged vector we can directly return the result.

Replacing the raw loops used for complete copy by range based loops or for_each
So instead of doing this
{% highlight cpp %}
for (int i = 0; i < (int)left.size(); i++)
    result.push_back(left[i]);
{% endhighlight %}
We can do this
{% highlight cpp %}
for(auto i : left) result.push_back(i);
{% endhighlight %}
{% highlight cpp %}
for_each(begin(left),end(left), [&result](int i){result.push_back(i);});
{% endhighlight %}
We can also vector constructor to perform the copy but for this our vector has to created at the time of copying which I don't want
or I don't know the proper way to do this <b>:( </b>. Using the <b> copy </b> algorithm, it will look like
{% highlight cpp %}
copy(begin(left), end(left),back_inserter(result));
{% endhighlight %}

Replacing the partial copy using raw loops with either for_each or copy
{% highlight cpp %}
 copy(begin(arr),begin(arr)+mid,back_inserter(left)); //Copies the last half
 //we can also move() but since it's smaller set not much gain
 copy(begin(arr)+mid,end(arr),back_inserter(left)); //Copies the right half
{% endhighlight %} 
This seems readable as copy copies in the way [begin,last) which makes it exclusive of last element and is picked up the right side
Also erasing the old vector after copying in merge() can help clear memory
We can make it more efficient by avoiding the copy and using move semantics to move the value saving temporary copies, although I am
not sure whether it helps or not. So, to every return statement add the <b>move()</b> function.

<strike> Autofy everything
My most favourite feature of C++ 11 is the auto which deduces the type using some smart moves, and it can drastically improve the
quality of the code but in C++ 11 signature of a function can't be autofied <b> :( </b> but C++14 overcomes this and allows
auto to be used in fuction as well as lambdas, C++14 can be enabled using <code> <b> -std=c++14 </b> </code> or <code> <b> -std=c++1y </b> </code>, for me sadly c++14 doesn't works
but the depreceated c++1y works. </strike>

Now code becomes
{% highlight cpp %}
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

void print(vector<int> v)
{
    auto echo = [&v](int i)
    {
        cout <<i<<" ";
    };
    for_each(begin(v),end(v),echo);
}

vector<int> merge(vector<int> left, vector<int> right)
{
    vector<int> result;
    while (not left.empty() or not right.empty())
    {
        if (left.size() > 0 and right.size() > 0)
        {
            if (left.front() <= right.front())
            {
                result.push_back(left.front());
                left.erase(left.begin());
            }
            else
            {
                result.push_back(right.front());
                right.erase(right.begin());
            }
        }
        else if (left.size() > 0)
        {
            move(begin(left),end(left),back_inserter(result));
            left.erase(begin(left),end(left));
            break;
        }
        else if (right.size() > 0)
        {
            move(begin(right),end(right),back_inserter(result));
            right.erase(begin(left),end(left));
            break;
        }
    }
    return move(result);
}

vector<int> mergeSort(vector<int> m)
{
    if (m.size() <= 1)
        return m;

    vector<int> left, right, result;
    int middle = m.size() / 2;

    copy(begin(m),begin(m)+middle,back_inserter(left));
    copy(begin(m)+middle,end(m),back_inserter(right));

    left = mergeSort(left);
    right = mergeSort(right);
    return move(merge(left, right));

}

int main()
{
    vector<int> v = {38,27,43,3,9,82,10};
    print(v);
    cout << "------------------" << endl;
    v = mergeSort(v);
    print(v);
}

{% endhighlight %}
which looks good and readable :)
However, it may or may not be the best or efficient of doing this, you have been warned.