Spet, 09
# cs246 Lec. 1
### Graphical shell 
 - launch programs
 - unteract with files/folders
 - user interface
 
### Command shell
 - text
 - launch programs
 - etc.

command prompt - win

terminal -mac

bash - Linux

### Commands
Examples
 - pwd - print work dir
 - cd - change dir
 - ls - list
 - wc - 
 - man - manual

Control signals
 - Ctr+D EOF
 - Ctr+C break

Input redirection 
 - cat filename - cat opens, reads, closes file
 - Cat - waits for input from shell (bash)
 - cat < input.txt -shell redirects input to the command
 - head number - opens and read first number lines of file

Output redirection
 - \> redirect output
 - it creates or overwrite a file
 - \>> append 

RegEx
 - \* wild card
 - ? any single char
 - [] terms

Redirection
 - stdin(<) -> program -> stdout(>)/stderr(2>)

## Pipes
taking output from one program using as input for another

use `|` to pipe, eg: 

    sort words | uniq | wc -w

