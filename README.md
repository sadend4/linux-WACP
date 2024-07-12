# WACP
## Installation
```
cd tools/perf
make
```
## Usage
Record performance data
```
perf record -k mono python sum.py
```
Inject JIT events
```
perf inject --jit --input perf.data --output perf.jit.data
```
Generate a performance analysis report
```
perf report -i perf.jit.data
```
## Example
sum.c
```
#include <stdio.h>

long long sum(int a, int b){
    int i,j;
    long long sum_value = 0;
    for (i = 0; i < a; i++) {
        for (j = 0; j < b; j++) {
        sum_value += i * j;
    }
  }

  return (long long)sum_value;
}
```
Compile to WebAssembly (WASM) code
```
emcc -g sum.c -o sum.wasm --no-entry -s EXPORTED_FUNCTIONS='["_sum"]'
```
sum.py
```
import wasmtime.loader
import wasmtime
import sys
import os
os.environ['PYTHONPERFSUPPORT'] = '1'


from wasmtime import Config, Store, Engine, Module, FuncType, Func, ValType, Instance

config = Config()

config.debug_info = True
config.cranelift_debug_verifier = True
config.profiler="jitdump"
config.strategy = "cranelift"

store = Store(Engine(config))
module = Module.from_file(store.engine, 'sum.wasm')
instance = Instance(store, module, [])

sum_func = instance.exports(store)['sum']

sum_value = sum_func(store,10000,10000)

print("The sum is", sum_value)
```
