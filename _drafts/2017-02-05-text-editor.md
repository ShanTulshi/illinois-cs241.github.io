---
layout: doc
title: "Text Editor"
permalink: text_editor
dueDates: "02/15 11:59pm"
---

## Backstory
You have just finished implementing a `Document` library and now your mentor
wants you to implement a text editor using your `Document` library. Since you've
already turned your `Document` library over to the QA team, who has run a <span
style="text-decoration:line-through">brutal</span> comprehensive set of tests
(and fixed any bugs they found), your mentor has decided to provide you with a
compiled version of the post-QA `Document` and `Vector` (as an "archive file")
for use in your text editor.

## Overview
Your editor can be run like any other editor with:

{% highlight bash %}
$ ./editor <filename>
{% endhighlight %}

The filename provided will be loaded into the `Document` for manipulation using
`Document_create_from_file()`. Then, editor_main.c reads commands from `stdin`.
When a command is read, the appropriate function in editor.c is called. You will
need to fill in the functions in `editor.c`.

**Important:** Lines will be 1 indexed for this assignment!

**Important:** Send all your debug printing to `stderr` instead of `stdout`.

Please keep the following in mind when implementing your text editor:

* Lines are 1 indexed (are they 1 indexed in the document? Be careful!)
* There is no limit on the number of lines
* There is no limit on the number of characters per line
* Some lines may be empty (you should treat these as "")
* Only when the user gives the save command will the contents of the file on
  disk be modified.
* All your editor operations should use the `Document` library to modify the
  document. Don't write directly to the file!

## Reference Implementation
In case some of the documentation does not go into detail on edge cases or if
you have questions on how your text editor should behave under certain
conditions, please refer to `editor_reference`.

`editor_reference` is an executable that the company stole (don't ask), which is
what your manager wants implemented, since the company does not have its source
code. For evaluation purposes `editor_reference` is the final say for how your
editor should behave.

## Format Library
We have provided a format library that handles all printing to `stdout`.  This
is to ensure that you do not lose points for not matching the format that the
autograder expects.

Please take a look at `format.c` and `format.h`. These files include all the
error messages and print outs that you need. (There is also an easter egg
function that might come in handy for multi line writes)

## Features
Make sure you can perform the following basic tasks from your editor program:

*   Display the contents of a file.
*   Write and append text to a file at a specified line number.
*   Delete text from a file at a specified line number.
*   Find text in a file and display the matching text with the line number.
*   Save the file to disk.
*   Quit the editor

## Display contents
Your text editor should be able to print out the content of the file/Document
that is loaded.

Your editor provides two print commands:

1.  print a single line
2.  print the whole Document

### print a single line
The user will provide the "p" command along with a line number when they want to
print out a single line from a file. It's also useful to get some context, so we
will print out the 5 lines above and below this line for them as well. If there
are not 5 lines above or below the requested line (say the line requested is
line 2), then print out as many lines as you can, up to 5.

To print the contents of the file `editor.h` using the editor, the user would
first open the file with the editor:

{% highlight text %}
$ ./editor editor.h
{% endhighlight %}

Then, to print out whatever is on the 30th line of this file, type:

{% highlight bash %}
p 30
{% endhighlight %}

Then hit enter.

The editor will now print out line 30 of the file (including five lines above
and below for context). (this is done by calling `handle_display_command()` in
`editor.c` with the string "p 3" as `command`).

{% highlight text %}
$ ./editor editor.h
p 30
25	 * based on the command passed in.
26	*/
27	void handle_append_command(Document *document, const char *command);
28
29	/**
30	 * Handles deleting from the document
31	 * based on the command passed in.
32	*/
33	void handle_delete_command(Document *document, const char *command);
34
35	/**
{% endhighlight %}

Suppose we had a file `things_on_my_table.txt` which contained the following:
{% highlight text %}
mug
salt
T.V. remote
{% endhighlight %}

(since I am very clean there are only three things on my table)

If we try to print out line 3 of this file, the printout will include up to 5
lines above and below the line specified:

{% highlight bash %}
$ ./editor things_on_my_table.txt
p 2
1    mug
2    salt
3    T.V. remote
{% endhighlight %}

**Important:** the "p" command includes line numbers in it's print out. Make sure
to use `format.{c,h}` to print this line out. Remember that lines are 1 indexed,
but make sure to take a look at `format.c`.

### print the whole document
The user will provide the "p" command (with no line number specified) when they
wish to print the whole document.

For example (using the same file we used above):

{% highlight bash %}
$ ./editor things_on_my_table.txt
p
1    mug
2    salt
3    T.V. remote
{% endhighlight %}


Again, make sure to use `format.{c,h}` to print these lines out.

### Errors
If the user asks you to print a document which is empty (calls "p" with or
without a line number on an empty document), use `print_document_empty_error()`
in `format.{c,h}` to tell them they can't do that.

If a user asks you to print a specific line which does not exist, please call
`invalid_line()` in `format.{c,h}` to let them know.

## Writing and Appending Text
There are two write modes for your editor: "write" and "append".

### write command
The "w" command should **OVERWRITE** a line completely. For example:

{% highlight bash %}
w 3 I like cats!
{% endhighlight %}

This will **OVERWRITE** line 3 with "I like cats!".

### append command
"a" will **APPEND** it's argument to the end of a line. For example, if line
three contained "I like cats!", then the user ran this command:

{% highlight bash %}
a 3  I also like dogs.
{% endhighlight %}

(append the string " I also like dogs." to line three, notice the extra
space)

They should then be able to run the print command and see the following
output:

{% highlight bash %}
1   mug
2   salt
3   I like cats! I also like dogs.
{% endhighlight %}

Note: If the line a user is appending to is currently empty, that's fine!

Note: If your editor is asked to write or append to a line that does not exist
yet, then it should fill in the gap with empty lines. (How can you use
`Document` to make this easy?)

### muti-line writes or appends
An extra feature your editor needs to account for is multi-line writes. For
example, the user can input the following:

{% highlight bash %}
w 3 I like cats!$Dogs are alright.
{% endhighlight %}

This will OVERWRITE line 3 with "I like cats!".

Second, because there is a "$", this will ALSO create a NEW line (the new
line four) reading "Dogs are alright.".

The old line 4 and all subsequent lines after get shifted down. Both the "w"
and "a" modes may contain multiple dollar signs, and an additional line
should be inserted for each.

So, for a complete example:

{% highlight bash %}
$ ./editor things_on_my_table.txt
p
1   mug
2   salt
3   I like cats!
a 3  I also like dogs$But I prefer cats.
p
1   mug
2   salt
3   I like cats! I also like dogs
4   But I prefer cats.
{% endhighlight %}

## Deleting text
The "d" command should delete a single line from a file. For example:

{% highlight bash %}
$ ./editor things_on_my_table.txt
p
1    mug
2    salt
3    I like cats!
4    Dogs are alright.
d 3
p
1    mug
2    salt
3    Dogs are alright.
{% endhighlight %}

Notice that the line "Dogs are alright" shifted up. (Is there anything in
`Document` that might help you with this?)

### Errors
If a user attempts to delete a line which does not exist, let them know!  Call
`invalid_line` from `format.h`.

## Finding Text
Your text editor should be able to find all occurrences of a word and display
the occurrences of the word with the line number the word occurred on.

If your user inputs the following into standard input:

{% highlight bash %}
/like cats
{% endhighlight %}

Then your text editor should print all occurrences (the whole line) with their
line numbers in following format:

{% highlight text %}
3    I [like cats]!
{% endhighlight %}

You only need to surround the first occurrence of the search term in each line
with square brackets. Use the `print_search_line()` function in `format.h` for
this.

So, for another example, suppose I had a file `kitties.txt` with the content:
{% highlight text %}
I like cats
I like cats
Dogs are alright
I like cats
{% endhighlight %}

And I ran a search operation:

{% highlight text %}
$ ./editor kitties.txt
/cats
1	I like [cats]
2	I like [cats]
4	I like [cats]
{% endhighlight %}

## Saving Text
Your text editor should be able to save all of the changes you make to the
file on disk (otherwise, what's the point?)

The save command is just the character "s"

{% highlight bash %}
s
{% endhighlight %}

## Quitting Your Text Editor
Your text editor should be able to quit without memory leaks.

So if your user inputs the following into standard input:

{% highlight bash %}
q
{% endhighlight %}

Then your text editor should quit and free/destroy your `Document` (even if
there are changes the user has not saved).

## Awesome Scripts

Since the editor reads commands from standard in, we can create a file, fill
it with commands, then send those commands to the editor.

Suppose you think 241 is the best class ever and you want to write a script
that makes sure everyone else knows that too. Create a file containing the
following:

{% highlight bash %}
w 1 I LOVE 241
w 2 241 is my FAVORITE class at UIUC
w 3 I$LOVE$CS$241
s
q
{% endhighlight %}

Lets try running this two different ways. First, on a file that doesn't
exist:

{% highlight bash %}
$ ./editor new_file < script
p
1   I LOVE 241
2   241 is my FAVORITE class at UIUC
3   I
4   LOVE
5   CS
6   241
{% endhighlight %}

And on a file that already has some content:

{% highlight bash %}
$ ./editor editor.c < script
$ head editor.c
I LOVE 241
241 is my FAVORITE class at UIUC
I
LOVE
CS
241

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
{% endhighlight %}

Scripts like this might be able to help you debug. Perhaps you can impress your
mentor with a bash script (google it) that run some editor scripts over some
files you create, then uses the tool `diff` (check the man pages) to make sure
your editor did the right thing.

## Grading, Submission, and Other Details
Please fully read details on [Academic
Honesty](https://courses.engr.illinois.edu/cs241/#/overview#integrity).  These
are shared between all MPs in CS 241.

We will be using Subversion as our hand-in system this semester. Our grading
system will checkout your most recent (pre-deadline) commit for grading.
Therefore, to hand in your code, all you have to do is commit it to your
Subversion repository.

To check out the provided code for `text_editor` from the class repository,
go to your cs241 directory (the one you checked out for "know your tools")
and run:

{% highlight bash %}
svn up
{% endhighlight %}

If you run `ls` you will now see a `text_editor` folder, where you can find
this assignment! To commit your changes (send them to us) type:

{% highlight bash %}
svn ci -m "mp2 submission"
{% endhighlight %}

Your repository directory can be viewed from a web browser from the
following URL:
[https://subversion.ews.illinois.edu/svn/fa16-cs241/NETID/vector](https://subversion.ews.illinois.edu/svn/fa16-cs241/NETID/text_editor)
where NETID is your University NetID. It is important to check that the
files you expect to be graded are present and up to date in your svn copy.

### Only edit the following files

We will only use the files in this list when running the autograder

*   `editor.c`

## Compile and Run

Because we have provided `Document` and `Vector` as a precompiled archive
file, please make sure to work on this assignment on your student VM. We
can't say what will happen on any other machine when you try to compile the
assignment.

To compile the release version of the code run:

{% highlight bash %}
make
{% endhighlight %}

This will compile your code with some optimizations enabled. If you use a
debugger on the 'release' build, it will not be able to show you the original
source code, or line numbers, most of the time. Optimizations sometimes expose
some bugs in your code that would not show up when no optimizations are enabled,
but since optimizations tend to reorder your code while compiling, an optimized
version of your code is not optimal for debugging.

To compile your code in debug mode, run `make debug` instead of `make`

If you compile in release mode, you will an executable called `editor`. If you
compile in debug mode, you will get an executable call `editor-debug`

We have also provided a file `editor_test.c` where you can programmatically test
your editor. This compiles to `editor_test` and `editor_test-debug`
