#Writing .d.ts files
$When using an external JavaScript library, or new host API, you'll need to use a declaration file (.d.ts) to describe the shape of that library. This guide covers a few high-level concepts specific to writing definition files, then proceeds with a number of examples that show how to transcribe various concepts to their matching definition file descriptions.
$$当我们在使用一个外部的JavaScript库或是新的API时，我们需要用一个声明文件（.d.ts）来描述这个库的结构。本节指引将会涵盖我们在写这种定义文件（definition files）时会涉及到的一些高级理念，并辅以一些例子，来展示这些理念是如何通过定义文件中对应的描述实现的。

##Guidelines and Specifics
###Workflow
$The best way to write a .d.ts file is to start from the documentation of the library, not the code. Working from the documentation ensures the surface you present isn't muddied with implementation details, and is typically much easier to read than JS code. The examples below will be written as if you were reading documentation that presented example calling code.
$$写.d.ts文件最好的方式不是根据代码来写，而是根据文档来写。根据文档来写代码能够保证你要表达的东西不会被实现细节所影响。而且文档通常也比JS代码要容易理解。所以我们后面的例子都会通过以你正在读一个用文档来表达由代码组成的例子的情景来写。

###Namespacing
$When defining interfaces (for example, "options" objects), you have a choice about whether to put these types inside a module or not. This is largely a judgement call -- if the consumer is likely to often declare variables or parameters of that type, and the type can be named without risk of colliding with other types, prefer placing it in the global namespace. If the type is not likely to be referenced directly, or can't be named with a reasonably unique name, do use a module to prevent it from colliding with other types.
$$当你在定义接口时（比如说"options"对象），你可以决定是否要把这些类型放到一个模块中。你需要根据具体情况来做这个决定 -- 如果用户很可能会时常需要声明这个类型的变量或参数，并且需要它的命名不与其他类型冲突的话，那你大可把它放到一个全局命名空间中。如果这个类型很可能不需要被直接引用，或是不太适合以一个独特的名字来命名的话，那你应该把它放在模块内以避免与其他类型发生冲突。

###Callbacks
$Many JavaScript libraries take a function as a parameter, then invoke that function later with a known set of arguments. When writing the function signatures for these types, do not mark those parameters as optional. The right way to think of this is "What parameters will be provided?", not "What parameters will be consumed?". While TypeScript 0.9.7 and above does not enforce that the optionality, bivariance on argument optionality might be enforced by an external linter.
$$很多JavaScript的库会事先把一个函数作为参数，并在之后用获取到的参数来调用它。当我们在写这种类型的函数签名（function signatures）时，我们不应该把它们当作是可选参数。我们需要认真考虑"我们需要传入什么参数"而不是"我们要用什么参数"。虽然TypeScript 从0.9.7版本开始不再限制我们传入函数作为可选参数，我们仍旧可以通过外部工具在参数的可选性上强制进行双向协变（bivariance）。

###Extensibility and Declaration Merging
$When writing definition files, it's important to remember TypeScript's rules for extending existing objects. You might have a choice of declaring a variable using an anonymous type or an interface type:
$$当我们在写定义文件时，我们需要格外注意TypeScript在扩展已有对象时的规则。你可以用匿名类型或接口来声明一个变量：

**Anonymously-typed var**

```js
declare var MyPoint: { x: number; y: number; };
```

**Interfaced-typed var**

```js
interface SomePoint { x: number; y: number; }
declare var MyPoint: SomePoint;
```

$From a consumption side these declarations are identical, but the type SomePoint can be extended through interface merging:
$$从使用者的角度来说，这两中声明是等价的。但我们可以通过接口合并来扩展SomePoint类型：

```js
interface SomePoint { z: number; }
MyPoint.z = 4; // OK
```

$Whether or not you want your declarations to be extensible in this way is a bit of a judgement call. As always, try to represent the intent of the library here.
$$你需要根据实际情况来决定是否允许你的声明通过这种方式被扩展。从这里也可以显示出你对这个库的想法。

###Class Decomposition
$Classes in TypeScript create two separate types: the instance type, which defines what members an instance of a class has, and the constructor function type, which defines what members the class constructor function has. The constructor function type is also known as the "static side" type because it includes static members of the class.
$$TypeScript中的类会创建出两种独立的类型：一种是实例类型，它定义了类实例上的成员；另一种是构造函数类型，它定义了构造函数的成员。因为构造函数类型包括了类中的静态成员，我们可以把当作是"静态的"类型。

$While you can reference the static side of a class using the typeof keyword, it is sometimes useful or necessary when writing definition files to use the decomposed class pattern which explicitly separates the instance and static types of class.
$$虽然我们可以使用typeof关键字来引用一个类中的静态部分，但在写定义文件时，我们有时候需要用被解构过的类。它能明确地将类中的实例部分和静态部分分开。

$As an example, the following two declarations are nearly equivalent from a consumption perspective:
$$举个例子，从一个使用者的角度来看，下面的两种声明方式几乎是等价的：

**Standard**

```js
class A {
    static st: string;
    inst: number;
    constructor(m: any) {}
}
```

**Decomposed**

```js
interface A_Static {
    new(m: any): A_Instance;
    st: string;
}
interface A_Instance {
    inst: number;
}
declare var A: A_Static;
```

$The trade-offs here are as follows:
$$它们的不同之处在于：

$Standard classes can be inherited from using extends; decomposed classes cannot. This might change in later version of TypeScript if arbitrary extends expressions are allowed.
It is possible to add members later (through declaration merging) to the static side of both standard and decomposed classes
$$我们可以用extends继承标准的类，而不能集成解构过的类。除非下一个版本的TypeScript允许这种继承这种表达式。

$It is possible to add instance members to decomposed classes, but not standard classes
You'll need to come up with sensible names for more types when writing a decomposed class
$$我们可以给解构过的类添加新的实例成员，但不能给标准类添加。在写解构类的时候，如果你想要添加更多类型的话，你可能需要想一些更有意义的名字。

###Naming Conventions
$In general, do not prefix interfaces with I (e.g. IColor). Because the concept of an interface in TypeScript is much more broad than in C# or Java, the IFoo naming convention is not broadly useful.
$$总的来说，不要在接口前加I（比如：IColor）。因为TypeScript中接口的概念要比C#或Java更宽泛。IFoo这类命名习惯没有太多作用。

##Examples
$Let's jump in to the examples section. For each example, sample usage of the library is provided, followed by the definition code that accurately types the usage. When there are multiple good representations, more than one definition sample might be listed.
$$来看看下面的这些例子。每个例子都先给出了一个库的使用用法，然后再给出对于这样的库来说比较合适的类型定义。如果一个库有多种比较合适的定义方式的话，这里会把它们都列举出来。

###Options Objects
**Usage**

```js
animalFactory.create("dog");
animalFactory.create("giraffe", { name: "ronald" });
animalFactory.create("panda", { name: "bob", height: 400 });
// Invalid: name must be provided if options is given
animalFactory.create("cat", { height: 32 });
```

**Typing**

```js
module animalFactory {
    interface AnimalOptions {
        name: string;
        height?: number;
        weight?: number;
    }
    function create(name: string, animalOptions?: AnimalOptions): Animal;
}
```

###Functions with Properties
**Usage**

```js
zooKeeper.workSchedule = "morning";
zooKeeper(giraffeCage);
```

**Typing**

```js
// Note: Function must precede module
function zooKeeper(cage: AnimalCage);
module zooKeeper {
    var workSchedule: string;
}
```

###New + callable methods
**Usage**

```js
var w = widget(32, 16);
var y = new widget("sprocket");
// w and y are both widgets
w.sprock();
y.sprock();
```

**Typing**

```js
interface Widget {
    sprock(): void;
}

interface WidgetFactory {
    new(name: string): Widget;
    (width: number, height: number): Widget;
}

declare var widget: WidgetFactory;
```

###Global / External-agnostic Libraries
**Usage**

```js
// Either
import x = require('zoo');
x.open();
// or
zoo.open();
```

**Typing**

```js
module zoo {
  function open(): void;
}

declare module "zoo" {
    export = zoo;
}
```

###Single Complex Object in External Modules

**Usage**

```js
// Super-chainable library for eagles
import eagle = require('./eagle');
// Call directly
eagle('bald').fly();
// Invoke with new
var eddie = new eagle(1000);
// Set properties
eagle.favorite = 'golden';
```

**Typing**

```js
// Note: can use any name here, but has to be the same throughout this file
declare function eagle(name: string): eagle;
declare module eagle {
    var favorite: string;
    function fly(): void;
}
interface eagle {
    new(awesomeness: number): eagle;
}

export = eagle;
```

###Callbacks
**Usage**

```js
addLater(3, 4, (x) => console.log('x = ' + x));
```

**Typing**

```js
// Note: 'void' return type is preferred here
function addLater(x: number, y: number, (sum: number) => void): void;
```
