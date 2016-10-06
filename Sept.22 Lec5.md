**Read ints from stdin and echo one per line**

```c++
int main() {
	int i;
  while(true) {
    cin >> i;
    if (cin.fail()) break;
    cout << i << endl;
  }
}
```

if cin fails on bad input 

- fail bit/
  - cin.fail() is true

if cin fails on EOF

- fail bit 
- EOF bit
  - cin.eof() true



**read ints simplified**

```c++
int main() {
  int i;
  while(true) {
    if (!(cin >> i)) break;
    cout << i << endl;
  }
}
```

```c++
int main() {
  int i;
  while(cin >> i) {
    cout << i << endl;
  }
}
```



**read ints and handle bad inputs**

```c++

```







# Stream Abstraction 

eg: file I/O using streams

c

```c
# include <stdio.h>
int main() {
  char s[256];
  FILE *file = fopen("suite.txt", "r");
  while(true) {
    fscanf(file, "%255s",s);
    if(feof(file)) break;
    printf("%s\n", s);
  }
}
```



c++ 

```c++
# include <iostream>
# include <fstream>
using namespace std;
int main() {
  ifstream file {"something.txt"}; //open file stream
  string s;
  while (file >> s) {
    cout << s << endl;
  }
}
```

