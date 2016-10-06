eg: Payday is on the last Friday of the month, compute this month's payday

syntax: ./payday



```shell
#!/bin/bash
answer(){
  if [ $1 -eq 31 ]; then
  	echo "This month: the 31st"
  else
  	echo "This month: the ${1}th"
  fi
}
answer $(cal | awk '{print $6}' | egrep [0-9] | tail -1)
```

`./payday` -- current month

`./payday march 2016` -- some other month

----

`./payday` -no cmd line args

​			$1 - date

​			$2 - nothing



**functions cannot access command-line arguments **



# Testing

- essential part of software development

  - ongoing - not just at the end
  - begin *before* you start coding -- understand requirements and behaviours
  - should be building "test suits" -- set of tests that document expected behaviour
- testing is not debugging -- testing needs to be systematic and reproducible
- testing cannot guarantee correctness

  - can only prove "wrongness" -- ie. match expected behaviour
- ideally, you should have a separate tester. ie. dev/tester are different

  - bias exist, want it to be objective




### Human testing vs. machine testing

- Human testing 
  - people look at your code, find flaws
  - include code inspections, walkthroughs
- Machine testing
  - run program with selected input, check against expected output
    - check expected behaviour
    - cannot check everything - choose test cases carefully
  - type: black box/grey box/white box
    - black-box: testers have no knowledge of internals - 
    - grey-box: some limited access to implementation
    - white-box: full access to implementation/complete knowledge
- start with black-box
  - check various classes of input.  eg: numeric, +/-
  - boundaries of valid ranges ("edge cases"). min/max value
  - mult. simultaneous boundaries ("corner cases")
  - extreme cases - outside expectations - give binary file instead of int
- white-box testing to supplement
  - ensure execute all logical paths in a program (complete?) - add more test cases
  - make sure every function runs



# C++

invented in 1979 by Bjarne Stroustrup

- working with simula (00 lang)
- add 00 features of similar to C
- "C with Classes" C++ 1983
- C++14 standards are used. C++17 is on its way

```c
# include <stdio.h>
int main() {
  prinf("Hello world \n");
  return 0;
}
```

```c++
# include <iostream>
using namespace std;
int main() {
  cout << "Hello world" << endl;
  return 0;
}
```



- main must return an `int` in C++
- c++ I/O -uses a header `<iostream>`
- output:
  - `std::cout << ... << ... << std::endl`
- `namespace` is a way of grouping (functionality)
- `using namespace std;` lets us say `cout`/`endl` without say `std::cout` / `std:: endl`
- return statement -returns a status code ($?)
  - if return statement is left out, main will return 0;

### Compile 

In the student.cs env.

`g++-5 -std=c++14 program.cc -o program`



### Input / Output

- I/O streams
  - cout - printing to standard output (stdout)
  - cin - reading from standard in (stdin)
  - cerr - printing to standard error (stderr)
- I/O operators 
  - `<<` put to. eg: `cout << x;`
  - `>>` get from. eg: `cin >> y;`
- eg add two numbers

```c++
# include <iostream>
using namespace std;
int main() {
  int x,y;
  cin >> x >> y;
  cout << x+y << endl;
}
```



- `cin` ignores whitespace 
- `cin >> x >> y` get two integer from cin assign to x,y
- what if input does not contain integer? -- error
- what if input terminates early (ctr+D) -- error
- we can check for this 
  - `cin.fail()` -- return true on failure of prev. operation
  - `cin.eof()` -- return true on EOF
  - if EOF is reached, both will be true but EOF is triggered after `fail()`

eg: read all ints from stdin and echo one per line stop on bad input or EOF

```c++
# include <iostream>
using namespace std;
int main() {
  int i;
  while (true) {
    cin >> i;
    if (cin.fail()) break;
    cout << i << endl;
  }
}
```



