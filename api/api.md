v8接口

主要命名空间



```
internal 内部接口
Array   数组的
ArrayBuffer
ArrayBufferView
还有各种变量的不列了
Handle 句柄 同local
HandleScope 记录所以指针
HeapObject 对象内存
```

看一下容易出问题的数组

arraybuffer

和arraybufferview不列了，直接截图了

```
Array 
	Length

定义
	Array* Array::Cast(v8::Value* value) {
#ifdef V8_ENABLE_CHECKS
  CheckCast(value);
#endif
  return static_cast<Array*>(value);
}
```

