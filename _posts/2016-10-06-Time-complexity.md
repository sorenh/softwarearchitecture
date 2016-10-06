---
layout: post
title:  "Time complexity - big-O notation"
date:   2016-10-06 12:02:21
tags:
  - academy
  - time complexity
  - complexity
  - big-O
---
This series is meant to be focused on practice, not so much on computer science theory. Time complexity and big-O notation are topics that typically get very theoretical, but I'll do my best to keep it down to Earth.

The "time complexity" of an algorithm tells us how much time will be spent solving a given problem. In particular, it helps us understand how much time will be spent solving a given problem as the input grows very large.

If you go through an array/list and perform some operation on each elements, it will take longer if you add more elements ot the array/list. If each element takes 1 second to process, and there are 10 elements, it'll take a total of 10 seconds. Add another 5 elements, and it'll take 15.

If you took a faster computer and did the same thing there, it might only take 0.5 seconds per element. When dealing with time complexity, we try to ignore this difference, so we just say that the time it takes depends on the number of elements. Let's call the number of elements "*n*". The simple loop we have (going through each element and performing a simple operation it) is said to be O(n).

Let's take another example.

A very naïve way to see if we have two identical elements in our list would be to go through each element and compare it to all the others.

    foreach index in 1..lengthOf(the_list):
        foreach comparison_index in 1..lengthOf(the_list):
            if index == comparison_index:
                skip   # Avoid comparing an element to itself
            if the_list[index] == the_list[comparison_index]:
                return True
    return False

For each element, we go through each element to compare. So if there are *n* elements, we would perform *n²* comparisons. We say that this algorithm is O(n²).

Let's try to optimise this a little bit. Let's imagine that the elements are large documents. Comparing them is expensive, so not only do we have to make many comparisons (*n²* of them!), each comparison could also be quite time consuming. To speed things up a little bit, let's first go through the list and calculate the md5sum of each element. Comparing those in the next step will be much, much faster.

    other_list = []
    foreach index in 1..lengthOf(the_list):
        md5sum_list[index] = md5sum(the_list[index])

    foreach index in 1..lengthOf(md5sum_list):
        foreach comparison_index in 1..lengthOf(md5sum_list):
            if index == comparison_index:
                skip   # Avoid comparing an element to itself
            if md5sum_list[index] == md5sum_list[comparison_index]:
                return True
    return False

So the first step involves going through a list and performing an operation (md5sum) on each element. We saw earlier that this is O(n).

The next step is our terrible comparison algorithm, which we saw was O(n²). The new, combined algorithm is *O(n+n²)*, but when we're using this notation, we discard the *n*, because as *n* grows, *n²* will grow much faster, and that's the only thing that matters. So you basically discard all but the worst element and we end up with an algorithm that is *O(n²)*.

## I already studied time complexity, why are you wasting my time here?

Some of you may have already heard about all of this in school. And, from a mathematical and computer science standpoint, it makes sense.

However, this site is not about computer science. It's about practical, scalable, software architecture.

Sit back for a second and look at the final algorithm up there. It's terrible, I know, but let's think it through.

How would the comparison actually work? Suppose they are large blobs of data.  Typically, you would compare the first byte of each file. Are they identical? Yes. Ok, move on the next byte. Are they identical? Yes. Ok, move on to the next one. Are they identical? No. Ok, so these elements are not the same. Done.

Three bytes was all we needed to look at. Sweet.

What if they actually were identical? Then we'd go through the entire file, comparing each byte and finally concluding that they are, indeed, identical.

Ok.

How does md5sum work? We don't have to go into much detail, but it's obvious that md5sum also goes through the whole file in order to reduce it to a 128 bit hash.

So what if we have 5 files, each 100 GB in size. Very, very different files. In fact, they're so different that the simple comparison would never need to move past the first 5 bytes to determine that they're not the same.

We'd have 5 runs of md5sum on some very large files. This could take 5 minutes a piece. That's 25 minutes. Then you'd make 5² = 25 very fast comparisons. Let's say they each take 1ms. That's 25 ms. According to the time complexity analysis (in particular the big-O notation), the first part of the algorithm doesn't even get counted, but at this input size, it clearly matters.

Let's see what the numbers look like for 1,000 files. It would take 5,000 minutes  to perform the md5sum. That's three and a half days. The comparisons would take 1,000² ms = 1000 seconds. That's around 15 minutes.

What about 10,000 files? 50,000 minutes for md5sum (35 days), 10,000² ms (1.1 days) for the comparison.

300,000 files turns out to be the crossover point. At 300,000 files, it takes 1042 days for the md5sums, and the same for the comparisons. More than 300,000 files and the comparisons start to take longer than the md5sums. Quite quickly, actually.

300,000 is a lot of files! Maybe many more than you know you'll ever have to deal with. Or much fewer. It depends. Very few things are one-size-fits-all.

The point is that time complexity as taught in schools does not hold all the answers.
