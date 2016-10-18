# Pipe

use output from one command as input for another command 

`cmd1 | cmd2 | cmd3 `

eg: 

`cat myfile | wc -w`

`cat words | sort | unique`

- can also use output from one command as parameter

`echo "abc"`



# RegEx

### Pattern matching

`*` - filename match any number of chars

- eg: ls *.txt

`?` - match single character 

- eg: ls sample?.txt

**Regular Expression**

- pattern for matching a string
- eg: web validation in JavaScript, text editor

### egrep (grep)

`egrep <condition> <file>` process the file one line at a time, print out lines matches the condition

eg: `egrep "cs246" index.html`, it will find lines contain string "cs246"

### regex rules

`()` to create a sub-expression

* eg: `(cs|CS)246` matches cs246 or CS246

  `(c|C)(s|S)246` matches cs246 or CS246 or cS246 or Cs246

`[]` match any one cha in brackets

- eg: `[cC][sS]246` cS246

  `[a-z][0-9]` can use for ranges

  `[^0-9]` single char *not* 0-9

  `[^A]`  *not* A

`.` - single character

`^` - start of file

`$` - end of file

### frequency of characters

`?` - 0 or 1 of preceding pattern. eg: `A?` 

`*` - 0 or more of 

`+` - 1 or more 



**eg:**

`egrep ".*" file.txt` any string

`egrep "^cs246" file.txt` match string "cs246" at start

`egrep "^.+$" file.txt`

`egrep "^(..)*$" file.txt` lines of even length



**sample questions:**

list files in curr dir whose name contains a single "a"

`ls | egrep "^[^a]*a[^s]*$"`

fetch words from global dict that start with "e" and consist of 5 chars

`egrep "^e....$"`



# File System

- files and directories structured in a tree

- top node is called root

- reference file / directories by their position in tree

  * eg: /u4/hello - absolute path specified relative to that

- every file can be specified with absolute path

- "working dir" 

  - directing that shell is using
  - can **point at** any directory
  - cd - other dir 
  - pwd - print curr dir

- **Relative path** - path to file or dir relative to working dir

  - for relative paths - `.` represent current dir and `..` represents parent dir

- | d                           | rwx rwx rwx     | 1          | h79sun | h79sun | 25    | Sept.9   | file.txt |
  | --------------------------- | --------------- | ---------- | ------ | ------ | ----- | -------- | -------- |
  | type: **-** file, **d** dir | permission bits | hard links | owner  | group  | bytes | mod date | name     |

In Unix, every file has 

- owner - primary user for this file (creator) and has all permissions

- group - set of one or more users with specific permissions

- | rwx          | rwx          | rwx   |
  | ------------ | ------------ | ----- |
  | user (owner) | group member | other |

  permission bits

- meanings of permission bits

| bit  | file                          | dir                                      |
| ---- | ----------------------------- | ---------------------------------------- |
| r    | file contents can be read     | dir contents can be read, eg: ls         |
| w    | file contents can be written  | dir contents can be modified, eg: create a new file |
| x    | file can be executed as a pro | dir contents can be navigated, eg: cd in to do "stuff" |


### chmod - set permissions on file or dir

|chmod|u|+|r|
------|
||g|-|w|