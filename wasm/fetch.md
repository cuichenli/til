# The easiest way to use fetch in WASM

Use the embedded JavaScript snippets. I have tried using the emscripten's fetch API, but it was not working in a waitable request. 
Was thinking to use WASI but conan does not have a great support for WASI yet. Then I found https://web.dev/articles/emscripten-embedding-js-snippets#em_async_js_macro.

Basically the code is like following:
```c++
EM_ASYNC_JS(emscripten::EM_VAL, do_fetch, (const char *url), {
    // TODO: fill in more detail
  url = UTF8ToString(url);
  const response = await fetch(url);
  return Emval.toHandle(await response.text());
});


void fetch(const FetchArgument &argument, FetchResult &result)
{
    emscripten::val response = emscripten::val::take_ownership(do_fetch( argument.url.c_str()));
    auto body = response.as<std::string>();
    *result.content = body;
    auto status_code = 200;
    *result.status_code = status_code;
}
```

Note: the functions used this async call, should be declared as an async function as the following:
```c++

void xxx() 
{
    fetch(..., ...);
}


EMSCRIPTEN_BINDINGS(interfaces) {
  emscripten::function("xxx", &xxx);
}
```

Then in JavaScript, you can now call:
```js
import M  from "./build/m.js"

M().then(async b => {
  await b.xxx()
})

```