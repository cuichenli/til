# Use conan to set environment for WASM

To set up environment for compiling to WASM with conan and CMake, we will need to create one conan profile, something like the following:
```
[settings]
os=Emscripten
arch=wasm
build_type=Release
compiler=gcc
compiler.cppstd=gnu17
compiler.libcxx=libstdc++11
compiler.version=12

[tool_requires]
emsdk/3.1.50
```
Then we can run the conan install command with this specific profile:
```
conan install . --output-folder=build --build=missing -pr:h <your-profile.path>
```


# Return value of WASM exported functions
Limited value types are allowed. https://stackoverflow.com/q/65429036


# Access file system from WASM
To access the physical file system from WASM, we need to compile it with `-s NODERAWFS=1`
https://stackoverflow.com/questions/57949390/how-to-do-file-inputs-via-node-js-using-emscripten#comment112415582_60510997