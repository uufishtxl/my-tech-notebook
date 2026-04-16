
### 修饰符：`Partial<T>`

* 作用：将所有的属性变成可选 (`?`)。

源码实现（伪代码）：
```TypeScript
type Partial<T> = {
	[P in keyof T]? : T[P]
}
```
### 修饰符：`Required<T>`

* 作用：将所有的属性变为必填

```TypeScript
type Required<T> = {
	[P in keyof T]-?: T[P];
}
```

>Q：为什么要用 `-?`
>A：TypeScript 的映射类型（Mapped Types）默认行为是“复读机”。如果不写 `-?` 表示的是原来什么样，我保持什么样。而 `+` 表示默认行为，通常忽略；`-` 表示移去，减除。

### 修饰符：`ReadOnly<T>`

将所有属性变成只读 (readonly)

```Typescript
// 给每个属性前面加上 readonly
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```
### 筛选：`Pick<T, K>`

* 作用：从 `T` 中挑选出一组属性 `K` 组成新类型。
* 场景：比如后端给了一个巨大的 `User` 对象，但你的组件只需要展示 `avatar` 和 `name`。
```TypeScript
// K extends keyof T：约束 K 必须是 T 里有的 Key，不能乱写
// [P in K]：只遍历你指定的那些 Key
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```
### 筛选：`Omit<T, K>`

* 作用：从 `T` 中提出掉 `K`，剩下的组成新的类型

```TypeScript
// 逻辑：先算出“剩下的Key” (Exclude)，再把它们 Pick 出来
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```
`Exclude` 是 `Omit` 的底层地基。它是用来做减法的。