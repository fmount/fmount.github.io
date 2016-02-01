---
layout: post
title: "Permutations, a simple Algorithm"
description: ""
category: 
tags: [algorithm]
comments: true
---

This post provide a simple method to calculate all the pemutations and it is, at the same time,
	a tribute to @antirez, the Redis creator, one of the greatest IT people that the world knows.

If you think the permutation algorithm is tricky to develop without a reference, the following is the trivial solution I found, if you read it one time you'll never forget it.


Let's assume you want the permutations of the following list:

	a b c d

Consider the list formed by just the first element:

	a

What are the permutations of 'a', of course they are just:

	a

Ok, now, take the next element, b. The permutations of a and b are the permutations of a modified to include b in every possible position, that is before a, and after a:

	(b) a
	a (b)

Take the next element. c. Insert c in very position of every permutation of 'a b':

	(c) b a
	b (c) a
	b a (c)

and, with the other permutation of 'a b'

	(c) a b
	a (c) b
	a b (c)

and you get all the six permutations of 'a b c', and so on. 

That is simple! :)

