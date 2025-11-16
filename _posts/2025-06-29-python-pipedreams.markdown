---
layout: post
title:  "Python pipedreams"
date:   2025-07-02 14:42:47 +0000
tags: python unix
---

Unix pipelines are a mainstay of any bioinformatics analysis. Similarly, python is an extremely powerful and flexible programming language that can be used to accomplish data processing tasks. Naturally there comes a point when you want to combine them. If you do this however, you will find that python has some idiosyncracies that make the iterative workflow of developing pipelines a bit of a pain. This post will show you how you can get around these idiosyncracies to realize your python pipedreams 😉.

## What are pipelines?

Unix pipelines are one of the most (if not the most) powerful feature of unix operating systems. They allow you to compose small modular building blocks to perform a huge variety of data manipulation tasks, and are particularly important in bioinformatics, where work often involves shuttling data between various programs and mashing data into shape. An example of a unix pipeline can be seen below for those who aren't familiar.


```sh
git ls-files | grep -Ev '.*.png|.*.jpg' | xargs files-to-prompt | clipcopy
```

Briefly this pipeline lists all the git tracked files in my current directory, filters out png and jpeg image files, collects these files into an argument list which is passed to files-to-prompt - a cli tool that concatenates text files into an llm friendly prompt format - and copies the output to my clipboard, ready for pasting into an AI chatbot. What makes this a unix pipeline is the presence of the `|` (pipe) symbol, which takes the standard output stream of one process, and connects it to the standard input of another. Since all these commands take text as input and produce text as output, they can be strung together to produce the desired output. This can be written to the clipboard as above, or it could be redirected to another file.

One of the really nice things about programming in this way is that it is highly interactive. The shell is a Read-Eval-Print-Loop (REPL), meaning every time we run a command, we can see the result. This is in contrast to the flow of writing a script, where we might write a chunk of code, write the file to disk, close it and then run it to see the result of our changes. This can be made easier by setting up a file watcher than runs your script on save, but nothing beats the rapid feedback you get from working in a REPL.

Pipelines are powerful in part because they expose a huge number of highly optimized built-in programs that you can use, dramatically reducing the amount of code (and bugs) you write. Sometimes however, you need to do some slightly more complex processing, and the built-in tools don't cut it. At this point you might create a custom python tool to slot into your pipeline. Although this works, if you pipe the output of the program to head, you may find that you generate a python exception, and an annoying stacktrace pops up in your terminal. We will get to why this happens later on, but first let me explain why you might want to do this.


## The iterative approach to pipeline development

You will often find yourself writing pipelines to process data you are not that familiar with. You may also have forgotten some of the command line arguments to a tool, or how the output tends to be formatted. For this reason the most effective way to create pipelines is to build them up piece by piece, checking the output as you go. Below is an example taken directly from my shell history.

```sh
  917  awk -F',' 'NF == 198 { print $0 }' headers.txt | head
  919  awk -F',' 'NF != 198 { print $0 }' headers.txt | wc -l
  921  awk -F',' 'NF == 198 { print $0 }' headers.txt | wc -l
  922  awk -F',' 'NF == 198 { print $0 }' headers.txt | tr ',' '\n' | sort | uniq -c | head
  923  awk -F',' 'NF == 198 { print $0 }' headers.txt | tr ',' '\n' | sort | uniq -c | grep -e '^563 ' | head
  924  awk -F',' 'NF == 198 { print $0 }' headers.txt | tr ',' '\n' | sort | uniq -c | grep -e '^\s+563 '
  925  awk -F',' 'NF == 198 { print $0 }' headers.txt | tr ',' '\n' | sort | uniq -c | grep -E '^\s+563 '
  926  awk -F',' 'NF == 198 { print $0 }' headers.txt | tr ',' '\n' | sort | uniq -c | awk '$1 != 563 { print $0 }'
  927  awk -F',' 'NF == 198 { print $0 }' headers.txt | tr ',' '\n' | sort | uniq -c | awk '$1 == 563 { print $0 }'
```

As you can see, I start with just the `awk` command and pipe it to `head` which shows me just the first few lines to check that the output matches my expectation. Later on I do the same thing with `grep` and realize from the output that what I needed was the `-E` flag not the `-e` flag. ending the pipeline with `head` as two nice features. Firstly it limits the amount of text printed to my terminal. If `headers.txt` was large, I might have been hit by a wall of text, which is unpleasant and makes it hard to look back through my shell history. The first few lines are enough to let me know if I am on the right track. The second less obvious benefit is that it short circuits the processing of the previous programs in the pipeline. By default `head` prints the first 10 lines, meaning the awk command will process 10 lines and then exit. If our file was 100,000 lines long we wouldn't want to process all of them just to look at the first 10! How does it accomplish this?

When head terminates (after the 10th line by default) it will close its standard in stream. The previous program attempts to write the next line of output to the standard in of `head` but finds that it can't because it is closed, which causes a 'broken pipe' (SIGPIPE) signal to be triggered. The default behaviour of most unix programs upon recieving this signal is to exit. This process repeats back through the pipeline until all programs have terminated. Although the default behavour of most unix programs is to exit gracefully on getting a SIGPIPE, python behaves differently by defualt. Lets confirm this by including a small python program in the above pipeline.

## Pythons interaction with SIGPIPE

I created a small script that does essentially what `tr` is doing in the above pipeline.

```python
#!/usr/bin/env python
# comma2nl.py
import sys

def main():
    for line in sys.stdin:
        sys.stdout.write(line.replace(",", "\n"))

if __name__ == "__main__":
    main()
```

If we run replace `tr` with `comma2nl.py` in our pipeline and pipe the result directly to head, we get a stacktrace in the output.

```sh
$ cat headers.txt headers.txt | awk -F',' 'NF == 198 { print $0 }' | ./comma2nl.py | head
sequence_id_heavy
sequence_heavy
locus_heavy
stop_codon_heavy
vj_in_frame_heavy
v_frameshift_heavy
productive_heavy
rev_comp_heavy
complete_vdj_heavy
v_call_heavy
Traceback (most recent call last):
  File "/home/user/wjs20.github.io/./comma2nl.py", line 12, in <module>
    main()
  File "/home/user/wjs20.github.io/./comma2nl.py", line 8, in main
    sys.stdout.write(line.replace(",", "\n"))
BrokenPipeError: [Errno 32] Broken pipe
```

This is because python does not exit cleanly by default on receiving a SIGPIPE signal, but raises a `BrokenPipeError` exception. This is undesirable, as it interferes with out iterative workflow by muddying out output with a stacktrace. luckily enough there is a straightforward fix. We can import the signals module from the standard library and assign the systems default action for SIGPIPE.

```python
import signal
signal.signal(signal.SIGPIPE, signal.SIG_DFL)
```

We can check the manpages for SIGPIPES default action. Which on my system is to Terminate.

'SIGPIPE      P1990      Term    Broken pipe: write to pipe with no readers; see pipe(7)'

If we modify our script with the line above and rerun it, we see that no exceptions are raised.

```sh
❯ cat headers.txt headers.txt | awk -F',' 'NF == 198 { print $0 }' | ./comma2nl.py | head
sequence_id_heavy
sequence_heavy
locus_heavy
stop_codon_heavy
vj_in_frame_heavy
v_frameshift_heavy
productive_heavy
rev_comp_heavy
complete_vdj_heavy
v_call_heavy
```

# Summary

Creating small custom tools in python for data processing pipelines can really improve your efficiency at the command line. In order to make python fit into the conventional streaming model of unix pipelines, just remember to set the default handler for SIGPIPE signals to the system defaults.
