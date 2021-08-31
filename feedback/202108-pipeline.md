# pipeline 提案讨论

由第七次JSCIG会议讨论整理

- 参与讨论：...
- 整理：@xiaoxiangmoe
- 修订：@hax

## hack pipeline operator

状态：

stage 0

问题：

1. Champions have tentative consensus for Hack-style as the way forward. 【存疑】
2. Dev community is still split, but there appears to be overwhelming support for pipeline syntax in whatever form we decide on. 【请求出处】
3. State-ofJS 调查显示，社区希望有 Pipeline Operators（第四希望的特性），但是并不清楚社区对几种 style 的偏好和倾向
4. 函数链式调用 demo

```ts
foo(bar(1, baz(x)[0]).method());
x |> baz(%)[0] |> bar(1,%).method() |> foo(%)
x |> x => baz(x)[0] |> y => bar(1,y).method() |> z => foo(z)
```

后面的写法可以提供更详细的有语义的参数名

6. async demo

```ts
let text = await (await fetch(...)).text();
let text = await fetch() |> await %.text();
let text = await (fetch() |> async response => (await response).text())
```

7. Static methods & Constructors (conversions)

```ts
let x =  Object.fromEntries(
  Object.entries(x).map(...)
);
let x = x |> Object.entries(%) |> %.map(...) |> Object.fromEntries(%);
let x = x |> Object.entries |> xs => xs.map(...) |> Object.fromEntries;
```

简洁程度接近。
后者大量用于 rxjs，ramda 等库

8. 无法复用函数的能力，比如解构，默认值等

```ts
const foo = bar |> ({ baz = '', qux = 0 } = {}) => baz.length + qux;
```

9. 不方便处理嵌套，能力残缺

```ts
const foo = x
  |> x => (x + z
    |> z => Math.min(100,z)
    |> z => Math.max(0, z))
  |> x => x + 1
```

如果用 hack style 那就没法用了

10. 现在 hack style 只支持 expression，在 do expression 出来之前，不方便使用循环，if else try catch 等 statement

11. 不方便在普通函数的上下文中，插入一个 async 函数上下文

```ts
function foo() {
  const url = '';
  const requestJob = url |> fetch |> async x => (await response).text();
  addTask(requestJob);
}
```

12. Vue 2 的 filter 和 Angular 的 pipe 都是 F# style 的。因为它们的影响，社区更习惯 F# style 的 pipeline operator。 Vue 3 把 F# style pipeline operator 当成了 filter 的[替换方案](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0015-remove-filters.md)。

> There is currently a stage 1 proposal for adding the Pipeline Operator to JavaScript, which provides largely similar syntactical convenience:
