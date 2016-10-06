# Permissions

| rwx  | rwx  | rwx  |
| ---- | ---- | ---- |
| u    | g    | o    |

- file permissions- apply to the file
- dir permissions - apply to dir **contents**
- what if they conflict?
  - will follow the most restrictive set of permissions



# Shell Scripts

### scripts programming

- File containing a set of **shell commands **, executed as programs

#### first. bash

- `#!/bin/bash`   -- can be used to print date, current user, work dir, etc. "she-bang"
  - have to make sure that the file is executable (`chmod u+x <filename>`)

#### Variables

- `x=1` note: without space chars. command sets x=1
- `echo $x` prints value of x
- `echo ${x}` avoids ambiguity, such as `${abc}` and `${a}bc`
- Variables are strings
- `" "` bash expands variables in double quotes: eg: `${PATH}`expands
- `' '` single quote are used for literal

#### Special variables in a script

- do not apply to command line
  - `$1` - represents first command line arg
  - `$2` - represents second command line arg
  - `$0` - name of script
  - `$?` - status of last executed program



**Examples**

check if a word is in the dictionary

```bash
./isItAWord word
isItAWord.bash
#!/bin/bash
egrep '^$1$' /user/share/dict/words
```



check if a word is a "good" password

```shell
if [ condition ] ; then
	...
elif
	...
else
	...
fi
# require space around square brackets []
```



```shell
#!/bin/bash
egrep '^$1$' /user/share/dict/words > /dev/null # /dev/null through away stdout
if [ $? -eq 0 ] ; then
	echo "Bad password"
else
	echo "Good password"
fi
```



Verify number of arguments

```shell
#!/bin/bash
usage() {
  echo "usage: $0 password"
}
if [ $# -ne 1 ] ; then
	usage
	exit
fi
```



## Variables in bash are strings

- can be manipulated as integers or strings

- choice of operator tells bash how to process

- | as integers                 | as strings         |
  | --------------------------- | ------------------ |
  | `-eq` equals                | `=` equals         |
  | `-ne` not equals            | `!=` not equals    |
  | `-lt` less than             | `<` "less than"    |
  | `-gt` greater than          | `>` "greater than" |
  | `-le` less than equal to    |                    |
  | `-ge` greater than equal to |                    |



## Loops

- eg: print numbers from 1 to $1     `./count 10`

```shell
#!/bin/bash
x=1
while [ $x -le $1 ]; do
	echo $x
	x=$((x+1)) 
done
```

eg: iterating over a list (of strings, separated by spaces)

- rename a group of files from .cpp tp .cc

```shell
#!/bin/bash
for name in *.cpp; do
	mv ${name} ${name%cpp}cc
done
```

eg: how many times a word($1) occurs in a file (\$2)

```shell
#!/bin/bash
x=0
for word in $(cat $2); do
	if [ $word = $1 ]; then
		x=$((x+1))
	fi
done
echo $x
```

```shell
#!/bin/bash
for j in $(ls); do
	echo "item $j"
done
```
