# C++ tips

## C++ split

```C++
#include <sstream>
#include <string>
stringstream str("bla,bla,asdd,asdfgf");
string s;
vector<string> res;

while (getline(str, s, ' ')) {
    res.push_bacl(s);
    cout << s << endl;
}
```
