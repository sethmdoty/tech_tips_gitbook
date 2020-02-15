# Automate Sysadmin Tasks with Python’s os.walk Function

_Using Python’s os.walk function to walk through a tree of files and directories._

I’m a web guy; I put together my first site in early 1993. And so, when I started to do Python training, I assumed that most of my students also were going to be web developers or aspiring web developers. Nothing could be further from the truth. Although some of my students certainly are interested in web applications, the majority of them are software engineers, testers, data scientists and system administrators. 

This last group, the system administrators, usually comes into my course with the same story. The company they work for has been writing Bash scripts for several years, but they want to move to a higher-level language with greater expressiveness and a large number of third-party add-ons. \(No offense to Bash users is intended; you can do amazing things with Bash, but I hope you’ll agree that the scripts can become unwieldy and hard to maintain.\) 

It turns out that with a few simple tools and ideas, these system administrators can use Python to do more with less code, as well as create reports and maintain servers. So in this article, I describe one particularly useful tool that’s often overlooked: os.walk, a function that lets you walk through a tree of files and directories. 

#### os.walk Basics <a id="os.walkbasics"></a>

Linux users are used to the `ls` command to get a list of files in a directory. Python comes with two different functions that can return the list of files. One is `os.listdir`, which means the “listdir” function in the “os” package. If you want, you can pass the name of a directory to `os.listdir`. If you don’t do that, you’ll get the names of files in the current directory. So, you can say: 

```text
In [10]: import os
```

When I do that on my computer, in the current directory, I get the following: 

```text
In [11]: os.listdir('.')
Out[11]:
['.git',
 '.gitignore',
 '.ipynb_checkpoints',
 '.mypy_cache',
 'Archive',
 'Files']
```

As you can see, `os.listdir` returns a list of strings, with each string being a filename. Of course, in UNIX-type systems, directories are files too—so along with files, you’ll also see subdirectories without any obvious indication of which is which. 

I gave up on `os.listdir` long ago, in favor of `glob.glob`, which means the “glob” function in the “glob” module. Command-line users are used to using “globbing”, although they often don’t know its name. Globbing means using the \* and ? characters, among others, for more flexible matching of filenames. Although `os.listdir` can return the list of files in a directory, it cannot filter them. You can though with `glob.glob`: 

```text
In [13]: import glob

In [14]: glob.glob('Files/*.zip')
Out[14]:
['Files/advanced-exercise-files.zip',
 'Files/exercise-files.zip',
 'Files/names.zip',
 'Files/words.zip']
```

In either case, you get the names of the files \(and subdirectories\) as strings. You then can use a `for` loop or a list comprehension to iterate over them and perform an action. Also note that in contrast with `os.listdir`, which returns the list of filenames without any path, `glob.glob` returns the full pathname of each file, something I’ve often found to be useful. 

But what if you want to go through each file, including every file in every subdirectory? Then you have a bit more of a problem. Sure, you could use a `for` loop to iterate over each filename and then use `os.path.isdir` to figure out whether it’s a subdirectory—and if so, then you could get the list of files in that subdirectory and add them to the list over which you’re iterating. 

Or, you can use the `os.walk` function, which does all of this and more. Although `os.walk` looks and acts like a function, it’s actually a “generator function”—a function that, when executed, returns a “generator” object that implements the iteration protocol. If you’re not used to working with generators, running the function can be a bit surprising: 

```text
In [15]: os.walk('.')
Out[15]: <generator object walk at 0x1035be5e8>
```

The idea is that you’ll put the output from `os.walk` in a `for` loop. Let’s do that: 

```text
In [17]: for item in os.walk('.'):
    ...:     print(item)
```

The result, at least on my computer, is a huge amount of output, scrolling by so fast that I can’t read it easily. Whether that happens to you depends on where you run this `for` loop on your system and how many files \(and subdirectories\) exist. 

In each iteration, `os.walk` returns a tuple containing three elements: 

* The current path \(that is, directory name\) as a string.
* A list of subdirectory names \(as strings\).
* A list of non-directory filenames \(as strings\).

So, it’s typical to invoke `os.walk` such that each of these three elements is assigned to a separate variable in the `for` loop: 

```text
In [19]: for currentdir, dirnames, filenames in os.walk('.'):
    ...:     print(currentdir)
```

The iterations continue until each of the subdirectories under the argument to `os.walk` has been returned. This allows you to perform all sorts of reports and interesting tasks. For example, the above code will print all of the subdirectories under the current directory, “.”. 

#### Counting Files <a id="countingfiles"></a>

Let’s say you want to count the number of files \(not subdirectories\) under the current directory. You can say: 

```text
In [19]: file_count = 0

In [20]: for currentdir, dirnames, filenames in os.walk('.'):
    ...:     file_count += len(filenames)
    ...:

In [21]: file_count
Out[21]: 3657
```

You also can do something a bit more sophisticated, counting how many files there are of each type, using the extension as a classifier. You can get the extension with `os.path.splitext`, which returns two items—the filename without the extension and the extension itself: 

```text
In [23]: os.path.splitext('abc/def/ghi.jkl')
Out[23]: ('abc/def/ghi', '.jkl')
```

You can count the items using one of my favorite Python data structures, `Counter`. For example: 

```text
In [24]: from collections import Counter

In [25]: counts = Counter()

In [26]: for currentdir, dirnames, filenames in os.walk('.'):
    ...:     for one_filename in filenames:
    ...:         first_part, ext =
 ↪os.path.splitext(one_filename)
    ...:         counts[ext] += 1
```

This goes through each directory under “.”, getting the filenames. It then iterates through the list of filenames, splitting the name so that you can get the extension. You then add 1 to the counter for that extension. 

Once this code has run, you can ask `counts` for a report. Because it’s a dict, you can use the `items` method and print the keys and values \(that is, extensions and counts\). You can print them as follows: 

```text
In [30]: for extension, count in counts.items():
    ...:     print(f"{extension:8}{count}")
```

In the above code, `f strings` displays the extension \(in a field of eight characters\) and the count. 

Wouldn’t it be nice though to show only the ten most common extensions? Yes, but then you’d have to sort through the `counts` object. It’s much easier just to use the `most_common` method that the `Counter` object provides, which returns not only the keys and values, but also sorts them in descending order: 

```text
In [31]: for extension, count in counts.most_common(10):
    ...:     print(f"{extension:8}{count}")
    ...:
.py     1149
        867
.zip    466
.ipynb  410
.pyc    372
.txt    151
.json   76
.so     37
.conf   19
.py~    12
```

In other words—not surprisingly—this example shows that the most common file extension in the directory I use for teaching Python courses is .py. Files without any extension are next, followed by .zip, .ipynb \(Jupyter notebooks\) and .pyc \(byte-compiled Python\). 

#### File Sizes <a id="filesizes"></a>

You can ask more interesting questions as well. For example, perhaps you want to know how much disk space is used by each of these file types. Now you don’t add 1 for each time you encounter a file extension, but rather the size of the file. Fortunately, this turns out to be trivially easy, thanks to the `os.path.getsize` function \(this returns the same value that you would get from `os.stat`\): 

```text
for currentdir, dirnames, filenames in os.walk('.'):
    for one_filename in filenames:
        first_part, ext = os.path.splitext(one_filename)
        try:
            counts[ext] +=
 ↪os.path.getsize(os.path.join(currentdir,one_filename))
        except FileNotFoundError:
            pass
```

The above code includes three changes from the previous version: 

1. As indicated, this no longer adds 1 to the count for each extension, but rather the size of the file, which comes from `os.path.getsize`.
2. `os.path.join` puts the path and filename together and \(as a bonus\) uses the current operating system’s path separation character. What are the odds of a program being used on a Windows system and, thus, needing a backslash rather than a slash? Pretty slim, but it doesn’t hurt to use this sort of built-in operation.
3. `os.walk` doesn’t normally look at symbolic links, which means you potentially can get yourself into some trouble trying to measure the sizes of files that don’t exist. For this reason, here the counting is wrapped in a `try/except` block.

Once this is done, you can identify the file types consuming the greatest amount of space in the directory: 

```text
In [46]: for extension, count in counts.most_common(10):
    ...:     print(f"{extension:8}{count}")
    ...:
.pack   669153001
.zip    486110102
.ipynb  223155683
.sql    125443333
        46296632
.json   14224651
.txt    10921226
.pdf    7557943
.py     5253208
.pyc    4948851
```

Now things seem a bit different! In my case, it looks like I’ve got a lot of stuff in .pack files, indicating that my Git repository \(where I store all of my old training examples, exercises and Jupyter notebooks\) is quite large. I have a lot in zipfiles, in which I store my daily updates. And of course, lots in Jupyter notebooks, which are written in JSON format and can become quite large. The surprise to me is the .sql extension, which I honestly had forgotten that I had. 

#### Files per Year <a id="filesperyear"></a>

What if you want to know how many files of each type were modified in each year? This could be useful for removing logfiles or \(if you’re like me\) identifying what large, unnecessary files are taking up space. 

In order to do that, you’ll need to get the modification time \(`mtime`, in UNIX parlance\) for each file. You’ll then need to convert that `mtime` from a UNIX time \(that is, the number of seconds since January 1st, 1970\) to something you can parse and use. 

Instead of using a `Counter` object to keep track of things, you can just use a dictionary. However, this dict’s values will be a `Counter`, with the years serving as keys and the counts as values. Since you know that all of the main dicts will be `Counter` objects, you can just use a `defaultdict`, which will require you to write less code. 

Here’s how you can do all of this: 

```text
from collections import defaultdict, Counter
from datetime import datetime

counts = defaultdict(Counter)

for currentdir, dirnames, filenames in os.walk('.'):
    for one_filename in filenames:
        first_part, ext = os.path.splitext(one_filename)
        try:
            full_filename = os.path.join(currentdir,
 ↪one_filename)
            mtime =
 ↪datetime.fromtimestamp(os.path.getmtime(full_filename))
            counts[ext][mtime.year] += 1
        except FileNotFoundError:
            pass
```

First, this creates `counts` as an instance of `defaultdict` with a `Counter`. This means if you ask for a key that doesn’t yet exist, the key will be created, with its value being a new `Counter` that allows you to say something like this: 

```text
counts['.zip'][2018] += 1
```

without having to initialize either the `zip` key \(for counts\) or the `2018` key \(for the `Counter` object\). You can just add one to the count, and know that it’s working. 

Then, when you iterate over the filesystem, you grab the `mtime` from the filename \(using `os.path.getmtime`\). That is turned into a `datetime` object with `datetime.fromtimestamp`, a great function that lets you move from UNIX timestamps to human-style dates and times. Finally, you then add 1 to your counts. 

Once again, you can display the results: 

```text
for extension, year_counts in counts.items():
    print(extension)
    for year, file_count in sorted(year_counts.items()):
        print(f"t{year}t{file_count}")
```

The `counts` variable is now a `defaultdict`, but that means it behaves just like a dictionary in most respects. So, you can iterate over its keys and values with `items`, which is shown here, getting each file extension and the `Counter` object for each. 

Next the extension is printed, and then it iterates over the years and their counts, sorting them by year and printing them indented somewhat with a tab \(`t`\) character. In this way, you can see precisely how many files of each extension have been modified per year—and perhaps understand which files are truly important and which you easily can get rid of. 

