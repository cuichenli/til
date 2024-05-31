# readFileSync could returns buffer with some prefixes

When using `readFileSync` function to read a file, you will see the returned buffer has some magic
number prefix:

```
$ cat ./another.txt
1231231%

$ cat ./index.js

import { readFileSync } from "fs"

const io = readFileSync("./another.txt")
console.log(io.buffer, io.byteLength, io.byteOffset)

$ node ./index.js

ArrayBuffer {
  [Uint8Contents]: <2f 00 00 00 00 00 00 00 31 32 33 31 32 33 31 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ... 8092 more bytes>,
  byteLength: 8192
} 7 8
```

From the above output, we can see the prefix `2f 00 00 00 00 00 00 00` and the actual data started
at position `8`. So to convert it to ArrayBuffer:
```
io.buffer.slice(io.byteOffset, io.byteOffset + io.byteLength)
```

This is not happening when using `read` function.
```
$ cat ./index.js


import { openSync, readSync } from "fs"

const buffer = Buffer.alloc(1024);
const f = openSync("./another.txt")
readSync(f, buffer)
console.log(buffer)

$ node ./index.js
<Buffer 31 32 33 31 32 33 31 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ... 974 more bytes>
```
