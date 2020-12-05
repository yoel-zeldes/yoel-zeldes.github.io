Title: Avoid committing junk
Slug: avoid-committing-junk
Date: 2016-11-10 21:00
Tags: git
Summary: Have you ever accidentaly pushed temporary stuff to the remote git server? Then keep reading.
cover: images/avoid-committing-junk/cover.png

In the development process every developer writes stuff he doesn't intend to commit and push to the remote server,
e.g. debug prints. It happens to all of us every now and then: we forget to remove this temporary stuff before committing...

I solved this somewhat embarrassing situation using a simple approach: to every line I don't want to accidentally commit
I add the magic characters sequence *xxx*. This sequence can be in any part of the line: inside a comment, as a variable name,
as a function name, you name it. A few usage examples:

* Debug print: `print 'xxx reached this line'`.
* Variable used for debugging: `xxxCounter = 0`.
* Temporary function: `def xxxPrintDebugInfo():`.
* TODO which must be attended before committing: `# TODO: don't forget to refactor this function xxx`.

The way I implemented it is using git hooks, which is git's mechanism to fire off custom scripts when certain important actions occur.
I used the pre-commit hook for validating the commit's content.

Just create a file with the name `.git/hooks/pre-commit` with the following content:

	:::bash
	#!/bin/sh

	marks=xxx,aaa,asd
	marksRegex=`echo "($marks)" | sed -r 's/,/|/g'`
	marksMessage=`echo "$marks" | sed -r 's/,/ or /g'`
	if git diff --staged | egrep -q "^\+.*$marksRegex"; then
			echo "You forgot to remove a line containing $marksMessage. You can forecully commit using \"commit -n\""
			exit 1
	fi

1. `marks` contains the characters sequences which are not allowed to be committed.
2. `git diff --staged` shows the changes which will be committed. The changes pass through a regular expression
   that searches for any forbidden mark (using `egrep`).
3. If a forbidden mark is found, the script exits with an error code, causing the commit to fail. 

If you want to bypass the hook (e.g. you want to commit a binary file such as an image, which may contain a forbidden mark), you can `commit -n`.
