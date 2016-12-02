---
layout: post
title:  The Deeply OO Nature of Python Collections
category: Python, Algorithms, Object Oriented
description: What I learned while debugging
---


<!--description-->



I was "cracking" a coding interview problem last week, and learned a fundamental truth about mutability in python.  Maybe you already understand this fundamental truth, or maybe you're as naïve as I was last week.  To find out, see if you can tell why this code, which is supposed to **generate all subsets of a list**, fails miserably:

{% highlight python %}
def subset(fullset, subsets=[[]]):
	if len(fullset) > 0:
	    x = [fullset.pop()]
	    newsubsets = list(subsets)
	    for i in newsubsets:
	        i.extend(x)
	    return subset(fullset, newsubsets+subsets)
	else:
	    return subsets
{% endhighlight %}



If your first impulse it to try to write this with generators - good for you, that approach should lead to a more efficient program, but for this example, I am working with this implementation to illustrate a point about the object orientation of Python.  The question is not, *"What is a faster implementation of the code?"* Instead, the question is, *"Why doesn't this code work?"*  

I wrote the above code with pen and paper, and was confident that it would work.  The basic idea is that to generate a list of subsets, you start with an empty set as the first subset.  Then for each element in the list, add the result of extending it to the previous list of subsets.  So this is how I imagined it would work:


	fullset = [1,2,3,4]:
	1. [[]]
	2. [[], [4]]
	3. [[], [4], [3], [4, 3]]
	4. [[], [4], [3], [4, 3], [2], [4, 2], [3, 2], [4, 3, 2]]
	5. [[], [4], [3], [4, 3], [2], [4, 2], [3, 2], [4, 3, 2], [1], [4, 1], [3, 1], [4, 3, 1], [2, 1], [4, 2, 1], [3, 2, 1], [4, 3, 2, 1]]

	

In line 2, the program extended each element of line 1 (just an empty list) with the popped element (x) from `fullset`, and added that new list to the list from line 1.  In line 3, the same thing: 3 was added (extended) to each list from line 2, and then those new lists were added to the list from line 2.  The program recursively calls the subset method, until there are no more elements to pop from `fullset`. Line 5 has the full answer for `fullset = [1, 2, 3, 4]`


This approach should work, but not with the implementation above.  Instead, it produces this output for `fullset = [1, 2, 3, 4]`
	
	[[4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1], [4, 3, 3, 2, 2, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1]]

Pretty ugly, right?  
To figure out what was going on here, I added some print statements to see what `newsubsets` and `subsets` were equal to after I extended `newsubsets`.

{% highlight python %}
def subset(fullset, subsets=[[]]):
    if len(fullset) > 0:
        x = [fullset.pop()]
        newsubsets = list(subsets)
        for i in newsubsets:
            i.extend(x)
        **print(newsubsets, "newsubsets", subsets, "subsets")**
        return subset(fullset, newsubsets+subsets)
    else:
        return subsets
{% endhighlight %}

Here are the first few lines of output:

	[[4]] newsubsets [[4]] subsets
	[[4, 3, 3], [4, 3, 3]] newsubsets [[4, 3, 3], [4, 3, 3]] subsets
	[[4, 3, 3, 2, 2, 2, 2], [4, 3, 3, 2, 2, 2, 2], [4, 3, 3, 2, 2, 2, 2], [4, 3, 3, 2, 2, 2, 2]] newsubsets [[4, 3, 3, 2, 2, 2, 2], [4, 3, 3, 2, 2, 2, 2], [4, 3, 3, 2, 2, 2, 2], [4, 3, 3, 2, 2, 2, 2]] subsets 


***Why are they the same after I only mutate newsubsets?***  I made sure that newsubsets is not referring to the same list as subsets, that is why I used newsubsets = **list**(subsets).  I could also have used newsubsets = **subsets[:]** and the same behavior would occur.   See [http://henry.precheur.org/python/copy_list](http://henry.precheur.org/python/copy_list) and [http://www-inst.eecs.berkeley.edu/~selfpace/cs9honline/Q2/mutation.html](http://www-inst.eecs.berkeley.edu/~selfpace/cs9honline/Q2/mutation.html) for more information about copying lists.  I was doing this part correctly...


But as the print statement shows, the `i.extend(x)` seems to be mutating both lists, not just `newsubsets`. And the reason I made the list `newsubsets` was so that I can edit a copy of `subsets` (by extending it with 'x') without mutating the original.  Then I can add the edited list to the unedited list.

After some poking around, I came to the realization that I was mutating the lists inside the lists, and that this is fundamentally different than modifying the list itself.  Here is the distinction:

{% highlight python %}
a = [1, 2, 3, 4]
b = list(a)
b.append(5)
print(a)
>>> [1, 2, 3, 4]
{% endhighlight %}

vs:

{% highlight python %}
c = [[1, 2], [3, 4]]
d = list(c)
d[1].append(5)
print(c)
>>>[[1, 2], [3, 4, 5]]
{% endhighlight %}
 
Notice in the second example, c was mutated -- even though I only appended 5 to d!

Why the difference in behavior?  Because Python is fundamentally object oriented.  I didn't directly mutate c -- I mutated one of the lists that c contains.  And likewise, I didn't actually append 5 to d, I appended 5 to a list that both d and c contain.

---

## Moral / TLDR:

If you simply copy a list of lists, the child-lists are not copies -- they are the same objects in the original and copied-list.  So you can't edit the child-lists without mutating the original list of lists.  Actually, this goes for any collection of objects…

Unless you do something fancier, by making copies of the child-lists... which is what I ended up doing by using ["copy" from the python library](https://docs.python.org/2/library/copy.html#copy.deepcopy).
*For collections that are mutable or contain mutable items, a copy is sometimes needed so one can change one copy without changing the other. This module provides generic shallow and deep copy operations.*

{% highlight python %}
import copy

def subset(fullset, subsets=[[]]):
    if len(fullset) > 0:
        x = [fullset.pop()]
        **newsubsets = copy.deepcopy(subsets)**
        for i in newsubsets:
            i.extend(x)
        return subset(fullset, subsets+newsubsets)
    else:
        return subsets
{% endhighlight %}

The modified code worked, and I learned about the fundamental OO nature of Python.  Debugging can be frustrating, but sometimes the enlightenment hits hard!  Moral of the story: make sure to know if your collection's copy contains objects that weren't explicitly copied, and the consequences this can have if you mutate them.  If you need copies of the objects, try using Python's `copy` module.