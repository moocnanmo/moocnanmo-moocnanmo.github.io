---
title: TypeScript使用总结
cover: https://img-blog.csdnimg.cn/9e41c79a5d0345448479661be83e566a.png
description: 总结一下TypeScript知识点，实习用过一段时间，编码时候的告警确实能减少BUG隐患，大势所趋，可得好好学习。
categories: '烂笔头经验记录'
---

[官方文档](https://www.tslang.cn/docs/handbook/basic-types.html)

# 基本类型

## 普通类型

> 包括 boolean、number、string、undefined、null、any、unknown、void、never、object。

```tsx
// 几种特殊属性说明
// any：可以表示任何类型，并且变量可以正常使用（不建议）；
// unknown：可以表示任何类型，但是不论用来做什么都是不合法的；
// void：表示没有任何类型，值只能为 undefined 和 null；当函数没有返回值时 其返回值类型是 void。
function fn(): void {}
// never：表示一个不存在的类型；never与其他类型的联合后，为其他类型；
type Test = string | number | never  // 相当于 type Test = string | number
// object：表示非原始类型，也就是除 number，string，boolean 之外的类型；
```

## 数组类型

```tsx
// 2.1 基本使用 类型[]；如 number[]、string[]、any[]
const arr1: number[] = [1, 2, 3]
// 2.2 元组 [类型1, 类型2, ...]；如 [string, number]
const arr2: [string, number] = ['str', 3]
// 2.3 数组泛型 Array<类型>；如 Array<number>、Array<string>
const arr3: Array<number> = [1, 2, 3]
```

## 联合类型、交叉类型

**1 联合类型**

> **类型1 | 类型2**；表示取值可以为多种类型中的一种；
>
> 如 number | string、(number | string)[]、Array<number | string>。

**2 交叉类型**

> **接口1 & 接口2**；表示把几种类型合并起来。

```tsx
// 应用场景: 比如 Student 继承 Person 上的属性
interface Person { 
 name: string;
}
type Student = Person & { age: number }
// 必须同时存在对应类型的 name 和 age 才符合 Student 类型
const obj: Student = { 
 name: "lk";
 age: 18 
}
```

## 类型保护、类型断言

**1 类型保护**

> 通过 `typeof` 方法判断返回值的类型，通过返回值类型对返回值做进一步处理；最终返回 符合函数返回值类型的最终值。

```tsx
function getLength(arg: number | string): number {
 if(typeof arg === 'string') { 
  return arg.length 
    } else { 
        return arg.toString().length 
 }
}
```

**2 类型断言**

> 可以用来手动指定一个值的类型；使用类型断言来告诉 TS，我（开发者）比你（编译器）更清楚这个参数是什么类型，你就别给我报错了。

```tsx
// 方式一: <类型>值
// 方式二: 值 as 类型  tsx中只能用这种方式
function getLength(arg: number | string): number {
    const str = <string>arg
    if (str.length) {
        return str.length
    } else {
        const number = arg as number
        return number.toString().length
    }
}
```

## 类型推断、字面量类型

**1 类型推断**

> TS会在没有明确的指定类型的时候推测出一个类型；
> 情况一: 定义变量时赋值了, 推断为对应的类型；
> 情况二: 定义变量时没有赋值, 推断为any类型。

**2 字面量类型**

> 有时候，我们需要定义一些常量，就需要用到字面量类型；
> 比如: type Sex = '男' | '女'；
> 这样就只能从这些定义的常量中取值，乱取值会报错。

# 接口(Interface)

> 接口是对象的状态(属性)和行为(方法)的抽象(描述)。

**1 使用接口来定义对象的类型**

```tsx
// 可选属性(age): 在可选属性名字定义的后面加一个 ? 符号
// 只读属性(id): 在属性名前用 readonly 来指定只读属性
// 拓展: 判断该用 readonly 还是 const 是要作为变量还是作为属性使用，
//      作为变量使用的话用 const，若作为属性则使用 readonly。
interface IPerson {
 readonly id: number;
    name: string;
    age?: number;
}
```

**2 用接口表示函数**

> 接口可以描述函数(参数的类型与返回的类型)

```tsx
interface SearchFunc {
 (source: string, subString: string): boolean
}
```

**3 一个类可以实现多个接口(implements)**

```tsx
interface Foo1 { foo1(): string }
interface Foo2 { foo2(): number }
class Car2 implements Foo1, Foo2 {
 foo0() { alert('foo0') }
    foo1() { return 'string' }
 foo2() { return 123 }
}
```

**4 一个接口可以继承多个接口(extends)**

```tsx
interface Foo1 { foo1(): string }
interface Foo2 { foo2(): number }
interface NewFoo extends Foo1, Foo2 {}
```

# 类型别名(type)

## 类型别名

> 类型别名会给一个类型起个新名字，有时和接口很像，但是可以作用于原始值，联合类型，元组以及其它任何你需要手写的类型。

```tsx
type Name = string     // 基本类型
type arrItem = number | string  // 联合类型             
const arr: arrItem[] = [1, '2', 3]
    
type Person = { name: Name }
type Student = Person & { grade: number  }      // 交叉类型
type Teacher = Person & { major: string  } 
type StudentAndTeacherList = [Student, Teacher] // 元组类型
const list:StudentAndTeacherList = [  
 { name: 'lin', grade: 100 }, 
    { name: 'liu', major: 'Chinese' }
]
```

## type 和 interface

**1 相同点**

```tsx
// 1 都可以定义一个对象或函数;
interface Person1 { name: string }
const person1: Person1 = { name: 'lin' }
type Person2 = { name: string }
const person2: Person2 = { name: 'lin' }
    
// 2 都允许继承: 
interface PersonI { name: string }
type PersonT = { name: string }
// (1) interface 继承 interfacetype 使用 extends
interface StudentI1 extends PersonI { grade: number }
interface StudentI2 extends PersonT { grade: number }
// (2) type 继承 typeinterface 使用交叉类型 &
type StudentT1 = PersonI & { grade: number }
type StudentT2 = PersonT & { grade: number }

// const person: Student = { name: 'lin', grade: 100 }
```

**2 不同点**

```tsx
// 1.interface(接口) 是 TS 设计出来用于定义对象类型的，可以对对象的形状进行描述;
// 2.type 是类型别名，用于给各种类型定义别名，让 TS 写起来更简洁、清晰;
// 3.type 可以声明基本类型、联合类型、交叉类型、元组，interface 不行;
// 4.interface可以合并重复声明，type 不行。

// interface 重复声明不报错
interface PersonI { name: string }
interface PersonI { age: number }
const person: PersonI = { name: 'lin', age: 18 }
// type 重复声明会报错
type PersonT = { name: string }
type PersonT = { age: number } // err
```

# 类

> 访问修饰符: 用来描述类内部的属性方法的可访问性；
> **public**: 默认值, 公开的外部也可以访问；
> **private**: 只能类内部可以访问；
> **protected**: 类内部和其子类内部可以访问；
> **static**: 类的静态属性，只能被类本身访问，而不能被实例化的对象访问。

# 函数

**5.1 为参数添加参数类型和返回值类型**

        ```tsx
        function foo1(x: number, y: number): number { 
         return x + y
        }
        ```

**5.2 函数的完整类型**

        ```tsx
        let foo2: (x: number, y?: number) = number 
        = function(x: number, y: number): number {
            return x + y
          }
        ```

**5.3 TS中的可选参数**

> JavaScript 里，每个参数都是可传可不传的；没传参的时候，它的值就是 undefined，但是在`TypeScript` 我们需要在参数名后面使用 ? 实现可选参数的功能(比如上面的参数 y )

**5.4 函数重载**

> 函数名相同, 而形参不同的多个函数

# 泛型

## 普通类型和函数泛型

```tsx
// 普通类型
function foo1 (a: string, b: number): [string, number] {
 return [a, b]
}
console.log(foo1('abc', 123))

// 函数泛型
function foo2 <K> (a: K, b: number): [K, number] {
 return [a, b]
}
console.log(foo2 <string> ('abc', 123))

// 多个泛型参数
function foo3 <K, V> (a: K, b: V): [K, V] {
 return [a, b]
}
console.log(foo3 <string, number> ('abc', 123))
```

## 泛型接口

> 在定义接口时, 为接口中的属性或方法定义泛型类型；在使用接口时, 再指定具体的泛型类型

```tsx
// 泛型接口
interface IbaseCRUD <T> {
  data: T[];
  add: (value: T) => void;
}
// 定义引用类
class User {
  id?: number
  name: string
  age: number
  constructor (name: any, age: any) {
    this.name = name
    this.age = age
  }
}
// 定义使用类
class UserCRUD implements IbaseCRUD <User> {
  data: User[] = []
  add(user: User): void {
    user = {...user, id: Date.now()}
    this.data.push(user)
  }
}
const userCRUD = new UserCRUD()
userCRUD.add(new User('tom1', 12))
userCRUD.add(new User('tom2', 13))
console.log(userCRUD)
```

## 泛型类

> 在定义类时, 为类中的属性或方法定义泛型类型；在创建类的实例时, 再指定特定的泛型类型

```tsx
class Foo <T> {    // 泛型类
 value: T
 constructor(value: T) {
  this.value = value
 }
}
console.log(new Foo <number>(123))    // number类
console.log(new Foo <string>('abc'))  // string类
```

# 工具类型

## 索引类型

**1 `keyof`（操作符）**

> 用于获取某种类型的所有键，返回一个联合类型;

```tsx
interface IPerson { 
    name: string;
    age: number;
}
type Test = keyof IPerson // 'name' | 'age'
```

**2 T[K]（索引访问）**

> T[K] 表示接口 T 的属性 K 所代表的类型;

```tsx
interface IPerson {
 name: string;
    age: number;
}
let type1:  IPerson['name'] // string
let type2:  IPerson['age']  // number
```

**3 extends (泛型约束)**

> T extends U，表示泛型变量可以通过继承某个类型，获得某些属性。

```tsx
interface ILength {
 length: number
}
function printLength<T extends ILength>(arg: T): T {
 console.log(arg.length)
    return arg
}
```

## 映射类型

**7.2.1 in 操作符**

> 用来对联合类型实现遍历。

        ```tsx
        // 相当于
        // type Obj = { 
        //     name: string; 
        //     school: string;
        // }
        type Person = "name" | "school"
        type Obj = { 
            [p in Person]: string 
        }
        
        const obj: Obj = { 
            name: "lk", 
            school: "jgs" 
        }
        ```

**7.2.2 Partial < T >**

> 将 T 的所有属性映射为可选属性。

        ```tsx
        // 原理：1 [P in keyof T]遍历T上的所有属性
        //      2 ? 设置属性为可选
        //      3 T[P]设置类型为原来的类型
        type Partial<T> = { 
            [P in keyof T]?: T[P] 
        }
        
        // 示例
        interface IPerson { 
            name: string;
            age: number;
        }
        let p1: IPerson = { 
            name: 'lin', 
            age: 18 
        }
        type IPartial = Partial<IPerson>  // Partial 改造
        let p2: IPartial = { age: 18 }    // 内部属性均为可选 ?
        let p3: IPartial = {}
        ```

**3 `Readonly` < T >**

> 将 T 的所有属性映射为只读属性。

```tsx
// 原理
type Readonly<T> = { 
    readonly [P in keyof T]: T[P] 
}

// 示例
interface IPerson {
  name: string
  age: number
}
type IReadOnly = Readonly<IPerson>
let p1: IReadOnly = {
  name: 'lin',
  age: 18
}
p1.name = 'line' // 报错
```

**4 Pick**

> 用于抽取对象子集，挑选一组属性并组成一个新的类型。

        ```tsx
        // 原理：参数1 T 表示要抽取的目标类型对象
        //      参数2 存在一个约束, K 一定要来自 T类型 内部属性字面量的联合类型
        type Pick<T, K extends keyof T> = { 
            [P in K]: T[P] 
        }
        
        // 示例
        interface IPerson { 
         name: string, 
         age: number, 
         sex: string 
        }
        type IPick = Pick<IPerson, 'name' | 'age'>
        let p1: IPick = { 
            name: 'lin', 
            age: 18 
        }
        ```

**5 Record**

> 创建新的属性，Record 是一种非同态映射类型。

        ```tsx
        // 原理: 参数1 传入继承于 any 的任何值，作为新创建对象的属性字面量的类型(键)
        //      参数2 作为新创建对象的值
        type Record<K extends keyof any, T> = { 
            [P in K]: T 
        }
        
        // 示例
        interface IPerson { 
            name: string, 
            age: number 
        }
        type IRecord = Record<string, IPerson>
        let personMap: IRecord = {
         person1: { name: 'lin', age: 18 },
         person2: { name: 'liu', age: 25 } 
        }
        ```

**6 拓展**

> 同态映射类型：只作用于 obj 属性而不会引入新的属性，如 Partial、`Readonly`、Pick;
> 非同态映射类型: 会创建新的属性，如 Record

## 条件类型

>T extends U ? X : Y
>若类型 T 可被赋值给类型 U,那么结果类型就是 X 类型,否则就是 Y 类型
>
>Exclude 和 Extract 的实现就用到了条件类型

**1 Exclude**

> Exclude 意思是不包含，`Exclude<T, U>` 会返回 `联合类型 T` 中不包含 `联合类型 U` 的联合类型。

```tsx
// 原理：never表示一个不存在的类型；never与其他类型的联合后，为其他类型。
type Exclude<T, U> = T extends U ? never : T

// 示例
// 相当于 type Test = "b" | "c"
type Test = Exclude<'a' | 'b' | 'c', 'a'>
```

**2 Extract**

> Extract<T, U> 提取联合类型 T 和联合类型 U 的交集；并作为一个联合类型进行返回。

```tsx
// 原理：
type Extract<T, U> = T extends U ? T : never

// 示例
// 相当于 type Test = 'key1' | 'key2'
type Test = Extract<'key1' | 'key2' | 'key3', 'key1' | 'key2'>
```

## 其他工具类型

**1 Omit**

> Omit<T, U> 从类型 T 中剔除 U 中的所有属性（对象类型）。

```tsx
// 原理1：使用 Pick 实现
// Pick 用于挑选一组属性并组成一个新的类型，Omit 是剔除一些属性，留下剩余的。
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>
// 原理2：不使用 Pick 实现
type Omit2<T, K extends keyof any> = {
 [P in Exclude<keyof T, K>]: T[P]
}

// 示例
// 相当于
// type IOmit = {
//   name: string;
// }
interface IPerson {
 name: string,
 age: number
}
type IOmit = Omit<IPerson, 'age'>
```

**2 `NonNullable`**

> `NonNullable<T>` 用来过滤类型中的 null 及 undefined 类型。

```tsx
// 原理：never表示一个不存在的类型；never与其他类型的联合后，为其他类型。
type NonNullable<T> 
= T extends null | undefined ? never : T

// 示例
type T0 = NonNullable<string | number | undefined>; // string | number
type T1 = NonNullable<string[] | null | undefined>; // string[]
```

**3 Parameters**

> Parameters 获取函数的参数类型，将每个参数类型放在一个元组中。

```tsx
// 原理：在条件类型语句中，可以用 infer 声明一个类型变量并且对它进行使用。
//      1 Parameters 首先约束参数T必须是个函数类型；
//      2 判断T是否是函数类型，如果是则使用 infer P 暂时存一下函数的参数类型，
//        后面的语句直接用 P 即可得到这个类型并返回，否则就返回never
type Parameters<T extends (...args: any) => any> 
= T extends (...args: infer P) => any ? P : never

// 示例
type T1 = Parameters<() => string>  // []
type T2 = Parameters<(arg: string) => void>  // [string]
type T3 = Parameters<(arg1: string, arg2: number) => void> 
// [arg1: string, arg2: number]
```

**4 `ReturnType`**

> `ReturnType` 获取函数的返回值类型。

```tsx
// 原理：1 ReturnType首先约束参数T必须是个函数类型
//      2 判断T是否是函数类型，如果是则使用infer R暂时存一下函数的返回值类型，
//     后面的语句直接用 R 即可得到这个类型并返回，否则就返回any
type ReturnType<T extends (...args: any) => any> 
= T extends (...args: any) => infer R ? R : any

// 示例
type T0 = ReturnType<() => string>    // string
type T1 = ReturnType<(s: string) => void>   // void
```

## 类型体操的概念

> `TypeScript` 高级类型会根据类型参数求出新的类型，这个过程会涉及一系列的类型计算逻辑，这些类型计算逻辑就叫做`类型体操`。当然，这并不是一个正式的概念，只是社区的戏称，因为有的类型计算逻辑是比较复杂的。

# TS 声名文件

## declare

> 当使用第三方库时，很多三方库不是用 TS 写的，我们需要引用它的声明文件，才能获得对应的代码补全、接口提示等功能。

**1 在 TS 中直接使用 `Vue`（错误演示）**

```tsx
// err：找不到名称“Vue”
const app = new Vue({ // 报错
 el: '#app',
 data: {
  message: 'Hello Vue!'
 }
})
```

**2 使用 declare 关键字，相当于告诉 TS 编译器，这个变量（`Vue`）的类型已经在其他地方定义了，你直接拿去用，别报错；**

```tsx
// 定义类型
interface VueOption {
    el: string,
    data: any
}
// 使用 declare 关键字进行声名
declare class Vue {
    options: VueOption
    constructor(options: VueOption)
}
// 不报错
const app = new Vue({
 el: '#app',
 data: {
  message: 'Hello Vue!'
 }
})

// 注意：declare class Vue 并没有真的定义一个类，只是定义了类 Vue 的类型，
//      仅仅会用于编译时的检查，在编译结果中会被删除。它编译结果是：
// const app = new Vue({
//     el: '#app',
//     data: {
//         message: 'Hello Vue!'
//     }
// })
```

## `.d.ts`

> 通常我们会把声明语句放到一个单独的文件（`Vue.d.ts`）中，这就是声明文件，以 `.d.ts` 为后缀。
>
> 一般来说，ts 会解析项目中所有的 `*.ts` 文件，当然也包含以 `.d.ts` 结尾的文件。所以当我们将 `Vue.d.ts` 放到项目中时，其他所有 `*.ts` 文件就都可以获得 `Vue` 的类型定义了。

```jsx
// ├── src
// |  ├── index.ts
// |  └── Vue.d.ts
// └── tsconfig.json

// --------- src/Vue.d.ts
// 定义类型
interface VueOption {
    el: string,
    data: any
}
// 使用 declare 关键字进行声名
declare class Vue {
    options: VueOption
    constructor(options: VueOption)
}

// --------- src/index.ts
// 使用
const app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

## 使用三方库

> 当我们使用三方库的时候，是不是所有的三方库都要写一大堆 `decare` 的文件呢？
>
> 不一定，要看社区里有没有这个三方库的 TS 类型包（一般都有）。社区使用 `@types` 统一管理第三方库的声明文件，是由 `DefinitelyTyped` 这个组织统一管理的；比如通过 `npm` 安装 `lodash` 的类型包，只要安装了，就可以在 TS 里正常使用 `lodash` 。
>
> 当然，如果一个库本来就是 TS 写的，就不用担心类型文件的问题，比如 `Vue3`。

## 自己写声明文件

> 比如你以前写了一个请求模块 `myFetch`
>
> 现在新项目用了 TS 了，要在新项目中继续用这个 `myFetch`，你有两种选择：
>
> - 用 TS 重写 `myFetch`，新项目引重写的 `myFetch`
> - 直接引 `myFetch`，给它写声明文件