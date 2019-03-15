代码优化

因为js速度慢进行的优化jit

多次调用一个部分，可能会导致其被改为一种中间语言

```
var counter = 0;
function test(x, y) {
    counter++;
    if (counter < 1000000) {
        // do something
        
    }
    return 'jeri';
}
```

优化回滚

　　因为V8是基于AST直接生成本地代码，没有经过中间表示层的优化，所以本地代码尚未经过很好的优化。于是，在2010年，V8引入了新的编译器-Crankshaft，它主要针对热点函数进行优化，基于JavaScript源代码开始分析而非本地代码，同时构建Hydroger图并基于此来进行优化分析。

　　Crankshaft编译器为了性能考虑，通常会做出比较乐观和大胆的预测—代码稳定且变量类型不变，所以可以生成高效的本地代码。但是，鉴于JavaScript的一个弱类型的语言，变量类型也可能在执行的过程中进行改变，鉴于这种情况，V8会将该编译器做的想当然的优化进行回滚，称为优化回滚。

　　示例如下：

```
var counter = 0;
function test(x, y) {
    counter++;
    if (counter < 1000000) {
        // do something
        return 'jeri';
    }
    var unknown = new Date();
    console.log(unknown);
}
```

　　该函数被调用多次之后，V8引擎可能会触发Crankshaft编译器对其进行优化，而优化代码认为示例代码的类型信息都已经被确定。但，由于尚未真正执行到new Date()这个地方，并未获取unknown这个变量的类型，V8只得将该部分代码进行回滚。优化回滚是一个很耗时的操作，在写代码过程中，尽量不要触发优化该操作。



具体代码就不仔细分析了，一个编译器，配上一张图理解大体是什么。

```
compiler.cc里面 

bool Compiler::CompileOptimized(Handle<JSFunction> function,
                                ConcurrencyMode mode) {
  if (function->IsOptimized()) return true;
  Isolate* isolate = function->GetIsolate();
  DCHECK(AllowCompilation::IsAllowed(isolate));

  // Start a compilation.
  Handle<Code> code;
  if (!GetOptimizedCode(function, mode).ToHandle(&code)) {
    // Optimization failed, get unoptimized code. Unoptimized code must exist
    // already if we are optimizing.
    DCHECK(!isolate->has_pending_exception());
    DCHECK(function->shared()->is_compiled());
    DCHECK(function->shared()->IsInterpreted());
    code = BUILTIN_CODE(isolate, InterpreterEntryTrampoline);
  }

  // Install code on closure.
  function->set_code(*code);

  // Check postconditions on success.
  DCHECK(!isolate->has_pending_exception());
  DCHECK(function->shared()->is_compiled());
  DCHECK(function->is_compiled());
  DCHECK_IMPLIES(function->HasOptimizationMarker(),
                 function->IsInOptimizationQueue());
  DCHECK_IMPLIES(function->HasOptimizationMarker(),
                 function->ChecksOptimizationMarker());
  DCHECK_IMPLIES(function->IsInOptimizationQueue(),
                 mode == ConcurrencyMode::kConcurrent);
  return true;
}
检查是否编译过，然后检查能不能被编译，然后编译，失败的话InterpreterEntryTrampoline执行


```

```
BytecodeGraphBuilder记录graph
graph里面通过node作为图的节点

```

