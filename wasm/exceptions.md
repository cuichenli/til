# Handle exceptions

Sometimes, when exceptions were thrown from the C++ side, it would not be caught on JS side, as 
> By default, exception catching is disabled in Emscripten. 
>  -- https://emscripten.org/docs/porting/exceptions.html

Take the following code as an example:
```c++
#include <filesystem>
#include <iostream>

int main(int argc, char const *argv[])
{
    try {
        auto absolute_scope = std::filesystem::canonical("templates").string();
        std::cout << absolute_scope << std::endl;
    } 
    catch (std::exception &e)
    {
        std::cout << "errrrr" << std::endl;
    }
    return 0;
}

```

When normally compiled, it will show the following errors:
```
Aborted(native code called abort())
/home/user/personal/myproj/t.cjs:128
      throw ex;
      ^

RuntimeError: Aborted(native code called abort())
    at abort (/home/user/personal/myproj/t.cjs:661:11)
    at _abort (/home/user/personal/myproj/t.cjs:4038:7)
    at wasm://wasm/000b427a:wasm-function[1797]:0x262ae
    at wasm://wasm/000b427a:wasm-function[1792]:0x2625e
    at wasm://wasm/000b427a:wasm-function[1788]:0x26157
    at wasm://wasm/000b427a:wasm-function[16]:0x11ce
    at wasm://wasm/000b427a:wasm-function[14]:0x10a9
    at /home/user/personal/myproj/t.cjs:692:14
    at callMain (/home/user/personal/myproj/t.cjs:4999:15)
    at doRun (/home/user/personal/myproj/t.cjs:5049:23)
```

In this case, we will need to compile the project with the option `-fexceptions`
```
em++ ./test.cpp -o ./test.js -fexceptions
```