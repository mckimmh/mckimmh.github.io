---
layout: post
title: "Shell Scripting"
date: 2023-09-26 11:45:00 +0100
categories: jekyll update
---

Shell scripting refers to writing a sequence of commands, typically to run programs and manage files. Shell scripting is one of the skills that I most wish I had at the start of my PhD. In the context of scientific computing (and research in computational statistics in particular), shell scripts may be used to automate the process of running programs and analysing their results. For example, one could use a script to define the variables used for a particular program, run that program taking those variables as input, then use a second program to create plots to visualise the output of the first program. Breaking up the steps of an analysis in this way can help to reduce code repetition and thus make a directory of code easier to maintain and expand.

Linux is an open-source operating system. The Linux kernel controls all the hardware and software on the computer. Utilities perform standard functions such as controlling files and programs. The shell is a special interactive utility, which allows users to start programs, manage files and manage processes running on the system. Bash (Bourne Again SHell) is a popular shell. A shell script allows you to enter multiple commands. A shell script should always begins with

	#!/bin/bash

To run a script `my_script.sh`, first grant permission for the script to be run using

	chmod u+x my_script.sh

Then run the script using

	./my_script.sh

I'll now give some useful Bash commands. There's plenty of resources online for learning Bash and the purpose of this post is more to encourage researchers to learn and use it. So instead of trying to give a comprehensive treatment of Bash commands, I will write about just a few commands. I've chosen commands that together vaguely sound like "Do-Re-Mi-Fa-So-La-Ti", taking as theme the song "Do-Re-Mi" from The Sound of Music.

## Do - the do in for loops

A `for` loop is useful for iterating through a list of items and performing the same set of commands on all of them. The basic structure of a `for` loop is:
   
	for variable in list_of_items
	do 
		command1 $variable
	done

In the above, the `$` prefix is used to get the value of the variable. A `for` loop allows you to repeat the same set of commands. For example, `echo` may be used to print text, so to print the same text multiple times you could use:

	for i in 1 2 3
	do
		echo "Hello"
	done

If you want to repeat a command a large number of times, you can instead use a range:

	for i in {1..100}
	do
		echo "Hello"
	done

Unfortunately, this syntax does not allow you to use variables. For example, if `n` is a variable defined as 3 (using `n=3`, where it is important *not* to have spaces), then the following does *not* print "Hello" 3 times:

	for i in {1..$n}
	do
		echo "Hello"
	done

Instead, you would need to use the following syntax:

	for (( i=1; i<=n; i++ ))
	do
		echo "Hello"
	done

A `for` loop also allows you to iterate over a number of files and apply the same commands to each:

	for file in file1 file2 file2
	do
		command1 $file
	done

As an example from Monte Carlo research, suppose that you have generated batches of samples, stored in `x1.txt` and `x2.txt`. Then you could process these sample batches (for instance making trace plots or computing sample means) using an R or Python script and a Bash `for` loop.

## Re - grep

`grep` prints lines that match patterns. To search for a string in a file, use:

	grep "string" file

Option `-r` is useful: it searches all files in the specified directory and sub-directories recursively. For example:

	grep -r "string" .

recursively searches all files in the current directory (specified by `.`) and its sub-directories for "string". I have found `grep` useful when I decide to rename a class or function in a directory of code (although one should try to avoid doing this, as it is a hassle).

## Mi - mkdir

`mkdir` makes a directory. These are useful for structuring a programming project. For example, when using C++, you might have a `src` directory for source code and an `include` directory for header files.

## Fa - functions

You can use functions in shell scripts. As with any programming language, functions are useful for reducing code repetition. The general syntax is:

	function_name()
	{
		commands
	}

 It's also possible to define a function using:

	function function_name
	{
		commands
	}
 
To pass arguments to a function, write the arguments after the function name. The first argument passed is avaible as `$1`, the second as `$2` and so on. For example, the following function prints its first and second argument to a file specified by its third argument:

	print_to_file()
	{
		str=$1
		file=$2
		echo $str > $file
	}

	print_to_file Hello output.txt

The operator `>` is used to redirect output. In this case, it redirects output to a file `output.txt`, so that once the script is executed, this text file contains `Hello`. The operator `>` will overwrite the file if it already exists, or create the file if does not exist.

## So - source

`source` may be used in a script to read and execute another script. For example, suppose that there are two scripts, `script1.sh` and `script2.sh`, which have some variables in common. To avoid defining the same variables in two different places (and hence making it burdensome to change the variables later), you could define the variables in a script `define_variables.sh`, then read these variables into `script1.sh` and `script2.sh` using

	source define_variables.sh

## La - ls

`ls` lists the file in a directory. This command is very useful for navigating large directories with multiple subdirectories.

## Ti - time

`time` may be used to time programs. For example:

	time ./my_program.out

## Conclusion

Bash is very useful! I've only covered a few commands here, and there is no pattern to them apart from their relation to The Sound of Music song "Do-Re-Mi". Check out "Linux Command Line and Shell Scripting Bible" by Blum and Bresnahan, or online tutorials for a more comprehensive treatment.
