---
layout: post
title: "[g]VIM :%s /Search/Replace/g"
description: ""
category: vim
tags: [vim]
comments: True
---
Vi[M] is equipped with _Search and replace_, one of the most powerful things we can find in this editor;
for very large files or when using the 'hjkl/e/w/b/W/E' combos generates more entropy to our lives, this feature
can save the day.
Let's discuss it from a practical point of view starting from few examples they can explain themselves:

	:8,10 s/search/replace/g
	:%s /search/replace/g
	:%s /search/replace/gc  #Ask for confirmation


#### Searching for ...

We usually can search for a string and put the cursor at the end:

	/string/e

or we can put it N lines over/above:

	/string/-2
	/string/+2


To offset from the beginning of the string, add a /b or /s with the offset that you want. 
For example, to move **three characters** from the beginning of the search, you'd use 

	/string/s+3 or /string/b+3  #"s" for "start" or "b" for "begin." 

To count from the end of the string, use /e instead, so

	/string/e-3 

will place the cursor on the third character from the last character of the matched string.


#### Special Characters to be more efficient ...

1. **Dot**:in a search, a dot or period (.) will match any single character; do a quick search using '/.' [slash dot]
and you'll see that it matches, literally, everything --> (letters, numbers, whitespace, etc..).

2. **Begin or end of a line?**: ^ or $; for instance, to find any line in a bash script that begins with a comment, you could use ^#, but to 
find empty lines, just use ^$ which will match any line without any characters.

3. **Digit or Characters, the usual question**:The \\d operator will match any digit in a search, while the \\D operator will match any non-digit in a search. To match any uppercase character, use \\u, while \\l will match any lowercase character. Using \\U will match any non-uppercase character, and \\L will match any non-lowercase character.

Special characters can be escaped with the backslash character, so use \\$ to search for a dollar sign in your file, or \\^ to search for the caret, and so forth.

#### Quantity counts ...


The '*' quantifier matches 0 or more characters (It matches abc, abby and absolutely);

	/abc\*

The '+' quantifier matches 1 or more characters (It matches just abc and not abby and absolutely);

	/abc\\+

The '=' quantifier matches 0 or one characters

	/abc\\=

The '{N,M}' match N to M instances of the character or string, for example:

	\\{0,10}

The '{n}' matches the exact number of characters

	\\{n}


#### Let's be insensitive ...

GNOME, Gnome, and gnome are completely different

	:set ignorecase
	:set noignorecase

If you don't want to toggle case-sensitivity on and off all the time, you can just use the \\c and \\C modifiers. The \\c modifier tells Vim to be case-insensitive, and \\C tells Vim to be case-sensitive.

For example, to search backwards through the text, ensuring that the search is case-sensitive, use ?\\Cpattern , so ?\\CGNOME will only match GNOME, not gnome or Gnome. Of course, this works for forward searches as well, so you could use /\\CGNOME instead.

	:set ignorecase smartcase	

To avoid the use/abuse of \\[Cc] identifiers, we can use the _smartcase_ setting: if your search term has at least one capital letter, Vim will switch to case-sensitive; otherwise it will use case-insensitive search.


#### Ranges ...
To match SuSE or SUSE, you'd use /S[Uu]SE

	[a-j]	#match any character that's  from a to j;
	[^a-j]	#match any character that's not a through j;
	[A-Q]	#any capital letter from A to Q;
	[1-5]	#any digit from 1 to 5;

#### Matching one or more terms

Let's say you want to replace the strings Kylo or Vader with the more generic term Darkness.
Instead of running two searches, you can use branches, which are separated by a backslash and pipe character : **\\|**

	:%s/Kylo\\|Vader/Darkness/gc

You're not limited to two terms, either.
If you want to replace Luke, Rey, and Han with Force, you could use 

	:%s/Luke\\|Ray\\|Han/Force/gc.

Generally speaking, there are tons of way to make an optimized _search and replace_; tools like grep
with the quickfix can enhance the powerful of the search operation and you can save a lot of time
spent to perform these operations.
