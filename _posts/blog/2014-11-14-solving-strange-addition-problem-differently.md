---
layout: post
title: "Solving Strange Addition Problem Differently "
modified:
categories: blog
excerpt: "How I solved Strange additon problem in a strange way" 
tags: [hackerearth,c++11,programming]
keyowrds: hackerearth,c++11,STL,programming 
comments: true
date: 2014-11-14T13:25:20+05:30
---

Tommorrow is my Operating System practical, going through them I realize that they are not too tough, so gave them a try and moved on. I am not in a mood to study right now, then it hit me why not try competitive programming ( meet my old darling) , so I decided to solve one of the easy problem, it was [Strange additions](http://www.hackerearth.com/problem/algorithm/the-reversed-numbers/). Problem itself was quite easy.
<blockquote>
  Given two numbers. First, reverse them, add them, reverse them again and print the answer 
</blockquote>

You might me thinking how my blog post changed itself from a narrative to talking one, never mind, that's me, happens quite often in my writing. Plus dude, it's an easy problem, there is nothing special about it that it deserves a blog post. Yes, I know but remember I changed it strangely. So *pseudocode* was like 
 {% highlight bash %}
 Take two numbers
 Reverse the numbers 
 Add the numbers and store the result
 Reverse the number and Print it 
 {% endhighlight %}

 Well, problem is trivial and my language of choice is C++ (*modern C++*). So insteading of doing it the old school way, i.e , looping and reversing the integers manually, I tried to make the compiler do the heavylifting for me (STL's). Call me lazy or silly, instead of reversing the integers I converted them to strings and reversed the strings then converted the strings back to integers. I know that my answer is not perfect but Hey ! it works :) 
 So my algo was something like this
 {% highlight bash %}

 Take input to Strings
 Reverse the strings
 Convert back strings to integers and add them 
 Convert the Result back to String
 Reverse the Result 
 Convert the Result back to Integer and Print it 
 {% endhighlight %}

 Thankly, C++11 made the code smaller. For converting integers to string instead of using the *stringstream or sstream*, I used the C++11's [to_string](http://en.cppreference.com/w/cpp/string/basic_string/to_string) method. For reversing the string I used the generic *reverse()* defined in the algorithm library and for coverting String back to integer instead of using String's stoi I used C++11's [stoull](http://en.cppreference.com/w/cpp/string/basic_string/stoul)  (since I needed an unsigned long long integer). So here's my final code.
 My thought : That looks like a badly styled  functional paradigmed C++ :)
 Sumbmitted Code Link : [Hacker Earth](http://www.hackerearth.com/submission/986973/)
 {% highlight C++ %}

#include <iostream>
#include <string>
#include <algorithm>
using namespace std;
int main()
{
 ios_base::sync_with_stdio(false);
 cin.tie(false);
 int t;
 string a_t,b_t,final;
 cin>>t;
 while(t--)
    {
    	cin >>a_t >>b_t;
    	reverse(begin(a_t),end(a_t));
    	reverse(begin(b_t),end(b_t));
        final = to_string(stoull(a_t) + stoull(b_t));
        reverse(begin(final),end(final));
        cout<<stoull(final)<<"\n";
    }
}
 
 {% endhighlight %}

  

