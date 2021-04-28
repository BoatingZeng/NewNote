## never
* https://cloud.tencent.com/developer/article/1594872

作用：
1. 使用never避免出现新增了联合类型没有对应的实现，目的就是写出类型绝对安全的代码

```typescript
type Foo = string | number;

function controlFlowAnalysisWithNever(foo: Foo) {
  if(typeof foo === "string") {
    // 这里 foo 被收窄为 string 类型
  } else if(typeof foo === "number") {
    // 这里 foo 被收窄为 number 类型
  } else {
    // foo 在这里是 never
    const check: never = foo;
  }
}

// 万一后面修改了Foo的类型，但是忘记修改上面的函数，else分支里的foo会收窄为boolean，这样会导致编译错误，达到了防止错误的目的
type Foo = string | number | boolean;
```
