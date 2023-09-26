---
layout: post
title: "The Sound of Bash"
date: 2023-09-26 11:45:00 +0100
categories: jekyll update
---

Linux is an open-source operating system. The Linux kernel controls all the hardware and software on the computer. Utilities perform standard functions such as controlling files and programs. The shell is a special interactive utility, which allows users to start programs, manage files and manage processes running on the system. Bash (Bourne Again SHell) is a popular shell.

There are many tutorials on Bash commands, but none are themed around song "Do-Re-Mi" from "The Sound of Music". This blog post remedies that absence.

## Preliminaries

A shell script begins with

	#!/bin/bash

A variable is defined (for example) using: `n=3`. It's important that there are no spaces in the variable definition!

`echo` may be used to print text. To get the value of a variable, prefix the variable with `$`. For example:

	echo $n

You can also use `$` and curly brackets:

	echo ${n}

## Do - the do in for loops

A `for` loop is useful for iterating through a list of items and performing the same set of commands on all of them. The basic structure of a `for` loop is:
   
	for variable in list_of_items
	do 
		command1 $variable
	done

A `for` loop allows you to repeat the same set of commands, for example using:

	for i in 1 2 3
	do
		echo "Hello"
	done

If you want to repeat a command a large number of times, you can instead use a range:

	for i in {1..100}
	do
		echo "Hello"
	done

Unfortunately, this syntax does not allow you to use variables. For example, if `n` is a variable defined as 3, then the following does *not* print "Hello" 3 times:

	for i in {1..${n}}
	do
		echo "Hello"
	done

Instead, you would need to use the following syntax:

	for (( i=1; i<=n; i++ ))
	do
		echo "Bonjour"
	done

A `for` loop also allows you to iterate over a number of files and apply the same commands to each:

	for file in file1 file2 file2
	do
		command1 ${file}
	done

As an example from Monte Carlo research, suppose that you have generated batches of samples, stored in `x1.txt` and `x2.txt`. Then you could process these sample batches (for instance making trace plots or computing sample means) using an R or Python script and a Bash `for` loop.

## Re - grep

`grep` prints lines that match patterns. To search for a string in a file, use:

	grep "string" file

Option `-r` is very useful: it searches all files in the specified directory and sub-directories recursively. For example (`.` refers to the current directory):

	grep -r "string" .

recursively searches all files in the current directory and its sub-directories for "string". I have found `grep` useful when I decided to rename a class or function in a directory of code.

## Mi - mkdir

`mkdir` makes a directory. These are useful for structuring a programming project. For example, when using C++, you might have a `src` directory for source code and an `include` directory for header files.

## Fa - functions

You can use functions in shell scripts! As an example, the following function prints its first and second argument to a file specifie by its third argument:

	print_to_file()
	{
		a=$1
		b=$2
		file=$3
		printf "$a\n$b\n" > $file
	}

	print_to_file hello world output.txt

## So - source

`source` may be used in a script to read and execute another script.

## La - ls

`ls` lists the file in a directory.

## Ti - time

`time` may be used to time programs. For example:

	time ./my_program.out

## Conclusion

Bash is very useful! I've only covered a few commands here, and there is no pattern to them apart from their relation to the Sound of Music song "Do-Re-Mi". Check out online tutorials or some of the resources below for a more comprehensive treatment.

## References

Linux Command Line and Shell Scripting Bible; Blum and Bresnahan.
