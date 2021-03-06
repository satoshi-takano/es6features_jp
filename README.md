# ECMAScript 6 <sup>[git.io/es6features](http://git.io/es6features)</sup>

## まえがき
ECMAScript 6(以下ES6)は近々公開される新しいバージョンのECMAScript標準です。
2015年6月の承認を目標としています。ES6は、2009年に行われたES5の標準化以来初めての重要な言語仕様アップデートです。
ES6で取り込まれる機能は[各ブラウザで実装中](http://kangax.github.io/es5-compat-table/es6/)です。

ES6の完全な言語仕様は[こちら](https://people.mozilla.org/~jorendorff/es6-draft.html)を参照してください。

ES6では以下の新機能が追加されました。
- [arrows](#arrows)
- [classes](#classes)
- [enhanced object literals](#enhanced-object-literals)
- [template strings](#template-strings)
- [destructuring](#destructuring)
- [default + rest + spread](#default--rest--spread)
- [let + const](#let--const)
- [iterators + for..of](#iterators--forof)
- [generators](#generators)
- [unicode](#unicode)
- [modules](#modules)
- [module loaders](#module-loaders)
- [map + set + weakmap + weakset](#map--set--weakmap--weakset)
- [proxies](#proxies)
- [symbols](#symbols)
- [subclassable built-ins](#subclassable-built-ins)
- [promises](#promises)
- [math + number + string + object APIs](#math--number--string--object-apis)
- [binary and octal literals](#binary-and-octal-literals)
- [reflect api](#reflect-api)
- [tail calls](#tail-calls)

## ECMAScript 6 の新機能

### Arrows
`Arrows`は関数定義の簡略シンタックスで、`=>`という記述で定義できます。
C#言語、Java 8、CoffeeScriptのそれと似たようなものです。
式と文の両方をサポートしています。従来の`function`との違いとして、`this`をその関数に束縛する、という点が挙げられます。


```JavaScript
// 式
var odds = evens.map(v => v + 1);
var nums = evens.map((v, i) => v + i);
var pairs = evens.map(v => ({even: v, odd: v + 1}));

// 文
nums.forEach(v => {
  if (v % 5 === 0)
    fives.push(v);
});

// Lexical this
var bob = {
  _name: "Bob",
  _friends: [],
  printFriends() {
    this._friends.forEach(f =>
      console.log(this._name + " knows " + f));
  }
}
```

### Classes
Classesは、従来のプロトタイプベースオブジェクト指向パターンの糖衣構文です。
宣言的にクラスを定義でき、互換性（他の多くのクラスベースOOP言語と...かな?）に拍車をかけます。

```JavaScript
class SkinnedMesh extends THREE.Mesh {
  constructor(geometry, materials) {
    super(geometry, materials);

    this.idMatrix = SkinnedMesh.defaultMatrix();
    this.bones = [];
    this.boneMatrices = [];
    //...
  }
  update(camera) {
    //...
    super.update();
  }
  get boneCount() {
    return this.bones.length;
  }
  set matrixType(matrixType) {
    this.idMatrix = SkinnedMesh[matrixType]();
  }
  static defaultMatrix() {
    return new THREE.Matrix4();
  }
}
```

### Object リテラルの改良
- コンストラクションでプロトタイプ設定
- プロパティへの値の割り当てが短く書ける
- メソッド定義も短く書ける
- 親（`super`）の呼び出しができる
- プロパティ名を動的に作れる

これらはClass定義と同様にオブジェクト指向設計の利点をもたらします。

```JavaScript
var obj = {
    // __proto__
    __proto__: theProtoObj,
    // ‘handler: handler’ はこう書ける
    handler,
    // メソッド
    toString() {
     // Super 呼び出し
     return "d " + super.toString();
    },
    // プロパティ名を動的に組み立てる
    [ 'prop_' + (() => 42)() ]: 42
};
```

### Template Strings
Template stringsは文字列組み立ての糖衣構文です。
Perl, Python等のそれと類似したものです。文字列を任意に組み立てるためにタグを埋め込むことができ、インジェクション攻撃を回避できたり文字列から高水準のオブジェクトを組み立てられたりします。

```JavaScript
// 基本的な文字列作成リテラル
`In JavaScript '\n' is a line-feed.`

// 複数行文字列
`In JavaScript this is
 not legal.`

// 文字列の補完
var name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`

// Construct an HTTP request prefix is used to interpret the replacements and construction
GET`http://foo.org/bar?a=${a}&b=${b}
    Content-Type: application/json
    X-Credentials: ${credentials}
    { "foo": ${foo},
      "bar": ${bar}}`(myOnReadyStateChangeHandler);
```

### Destructuring
パターンマッチを使った代入ができます。`Array`や`Object`をサポート。
Destructuringはフェイルソフトで、普通のオブジェクトに対してのルックアップ(`foo["bar"]`)と同様に、値が見つからなかった時は`undefined`を示します。

```JavaScript
// 配列
var [a, , b] = [1,2,3];

// オブジェクト
var { op: a, lhs: { op: b }, rhs: c }
       = getASTNode()

// オブジェクトでの簡略表記
// binds `op`, `lhs` and `rhs` in scope
var {op, lhs, rhs} = getASTNode()

// 引数でも同様のことができます
function g({name: x}) {
  console.log(x);
}
g({name: 5})

// フェイルソフトなので、値が見つからない時は`undefined`になります
var [a] = [];
a === undefined;

// デフォルト値を指定することができます
var [a = 1] = [];
a === 1;
```

### Default + Rest + Spread
引数のデフォルト値を指定できます。
```JavaScript
function f(x, y=12) {
  // 第二引数が渡されなかったとき（または`undefined`が渡された時）`y`の値は12になります
  return x + y;
}
f(3) == 15
```
関数呼び出しの際の末尾に複数の実引数を渡した時に、それを配列として受け取れます。  
Restは、従来利用していた`arguments`を置き換えるものです。
```JavaScript
function f(x, ...y) {
  // `y`は配列
  return x * y.length;
}
f(3, "hello", true) == 6
```
```JavaScript
function f(x, y, z) {
  return x + y + z;
}
// 配列を個別の引数として展開して渡すこともできます
f(...[1,2,3]) == 6
```

### Let + Const
`let`を使うとブロックスコープの変数を宣言できます。新種の`var`みたいなものです。  
`const`は定数を宣言できます。一度値を代入したら再び新しい値を代入することはできません。

```JavaScript
function f() {
  {
    let x;
    {
      // xはブロックスコープに束縛されています
      const x = "sneaky";
      // 定数に値を再代入することはできないため、これはエラーになります
      x = "foo";
    }
    // これもエラー。このスコープで`x`は既に宣言されている
    let x = "inner";
  }
}
```

### Iterators + For..Of
Iteratorオブジェクトは、CLRのIEnumerableやJavaのIterableのような、独自のイテレーションを実装できます。  
カスタムイテレータは`for..of`でイテレーションします。
配列で実現する必要はなく、LINQに似た遅延実行パターンを可能にします。

```JavaScript
let fibonacci = {
  [Symbol.iterator]() {
    let pre = 0, cur = 1;
    return {
      next() {
        [pre, cur] = [cur, pre + cur];
        return { done: false, value: cur }
      }
    }
  }
}

for (var n of fibonacci) {
  // 1000までにしとく
  if (n > 1000)
    break;
  console.log(n);
}
```

Iterationは以下のインターフェースに基づいています。(説明のため[TypeScript](http://typescriptlang.org)のシンタックスを使用):
```TypeScript
interface IteratorResult {
  done: boolean;
  value: any;
}
interface Iterator {
  next(): IteratorResult;
}
interface Iterable {
  [Symbol.iterator](): Iterator
}
```

### Generators
Generatorsは`function*`、`yield`を使って、反復処理の記述を容易にします。  
`function*`で宣言された関数はGeneratorインスタンスを返します。Generatorsは`next`、`throw`を備えたイテレータのサブタイプです。  
 These enable values to flow back into the generator, so `yield` is an expression form which returns a value (or throws).

Note: `await`風な非同期プログラミングにも使えます。`await`についてはES7のプロポーザルを参照してください。

```JavaScript
var fibonacci = {
  [Symbol.iterator]: function*() {
    var pre = 0, cur = 1;
    for (;;) {
      var temp = pre;
      pre = cur;
      cur += temp;
      yield cur;
    }
  }
}

for (var n of fibonacci) {
  // 1000までにしとく
  if (n > 1000)
    break;
  console.log(n);
}
```

Generatorのインターフェースは以下

```TypeScript
interface Generator extends Iterator {
    next(value?: any): IteratorResult;
    throw(exception: any);
}
```

### Unicode
完全なUnicodeサポートのための互換性を保った変更。
文字列中で使えるUnicodeリテラルと、正規表現の`u`モード、および文字列処理のための新しいAPIを含みます。
これらの追加はJavaScriptで動くグローバルなアプリをサポートします。

```JavaScript
// same as ES5.1
"𠮷".length == 2

// new RegExp behaviour, opt-in ‘u’
"𠮷".match(/./u)[0].length == 2

// new form
"\u{20BB7}"=="𠮷"=="\uD842\uDFB7"

// new String ops
"𠮷".codePointAt(0) == 0x20BB7

// for-of iterates code points
for(var c of "𠮷") {
  console.log(c);
}
```

### Modules
言語レベルでのコンポーネント定義サポートで、AMD,CommonJSなどのポピュラーなJavaScriptモジュールローダーを体系化したものです。  
ランタイムでの挙動はホストで定義されたデフォルトローダに従います。  
暗黙的な非同期モデルを採用しており、要求されたモジュールが有効になるまでコードは実行されません。

```JavaScript
// lib/math.js
export function sum(x, y) {
  return x + y;
}
export var pi = 3.141593;
```
```JavaScript
// app.js
import * as math from "lib/math";
alert("2π = " + math.sum(math.pi, math.pi));
```
```JavaScript
// otherApp.js
import {sum, pi} from "lib/math";
alert("2π = " + sum(pi, pi));
```

ほかに、`export default`と`export *`という機能を含んでいます。

```JavaScript
// lib/mathplusplus.js
export * from "lib/math";
export var e = 2.71828182846;
export default function(x) {
    return Math.exp(x);
}
```
```JavaScript
// app.js
import exp, {pi, e} from "lib/mathplusplus";
alert("2π = " + exp(pi, e));
```

### Module Loaders
モジュールローダーは以下をサポートします。
- 動的読み込み
- 状態の隔離
- グローバル名前空間の隔離
- Compilation hooks
- Nested virtualization

デフォルトのモジュールローダーを設定することができ、また分離されたコンテキストに閉じてコードを読み込み、実行するために新しいローダーを構成することもできます。

```JavaScript
// 動的読み込み（`System`はデフォルトローダー）
System.import('lib/math').then(function(m) {
  alert("2π = " + m.sum(m.pi, m.pi));
});

// 新しいローダー作成によるサンドボックス化
var loader = new Loader({
  global: fixup(window) // replace ‘console.log’
});
loader.eval("console.log('hello world!');");

// Directly manipulate module cache
System.get('jquery');
System.set('jquery', Module({$: $})); // WARNING: not yet finalized
```

### Map + Set + WeakMap + WeakSet
よくあるアルゴリズムのための効率的なデータ構造。
WeakMapsはメモリリークのないobject-keyのテーブルです

```JavaScript
// Sets
var s = new Set();
s.add("hello").add("goodbye").add("hello");
s.size === 2;
s.has("hello") === true;

// Maps
var m = new Map();
m.set("hello", 42);
m.set(s, 34);
m.get(s) == 34;

// Weak Maps
var wm = new WeakMap();
wm.set(s, { extra: 42 });
wm.size === undefined

// Weak Sets
var ws = new WeakSet();
ws.add({ data: 42 });
// addされたオブジェクトはどこからも参照されていないため、setに保持されません。
```

### Proxies
Proxiesは提供されたオブジェクトの持つ全ての挙動を保ったまま、メソッド呼び出しをフックしたり、オブジェクトを可視化したり、ロギングやプロファイリング等に活用できます。

```JavaScript
// 普通のオブジェクトをプロキシする
var target = {};
var handler = {
  get: function (receiver, name) {
    return `Hello, ${name}!`;
  }
};

var p = new Proxy(target, handler);
p.world === 'Hello, world!';
```

```JavaScript
// 関数オブジェクトをプロキシする
var target = function () { return 'I am the target'; };
var handler = {
  apply: function (receiver, ...args) {
    return 'I am the proxy';
  }
};

var p = new Proxy(target, handler);
p() === 'I am the proxy';
```

ランタイムレベルのメタ操作全てに対して有効なトラップがあります。

```JavaScript
var handler =
{
  get:...,
  set:...,
  has:...,
  deleteProperty:...,
  apply:...,
  construct:...,
  getOwnPropertyDescriptor:...,
  defineProperty:...,
  getPrototypeOf:...,
  setPrototypeOf:...,
  enumerate:...,
  ownKeys:...,
  preventExtensions:...,
  isExtensible:...
}
```

### Symbols
Symbolsは新しく導入されたプリミティブ型で、オブジェクト状態のアクセス制御を可能にします。  
文字列、シンボルのどちらでもプロパティのキーとして利用できます。  
任意で指定できる`name`パラメータはデバッグで利用されます。（Symbolを同定するものではない）  
Symbolsはユニークですが、`Object.getOwnPropertySymbols`のようなリフレクションの機能を使った場合にはプライベートになりません。


```JavaScript
var MyClass = (function() {

  // module scoped symbol
  var key = Symbol("key");

  function MyClass(privateData) {
    this[key] = privateData;
  }

  MyClass.prototype = {
    doStuff: function() {
      ... this[key] ...
    }
  };

  return MyClass;
})();

var c = new MyClass("hello")
c["key"] === undefined
```

### Subclassable Built-ins
In ES6, built-ins like `Array`, `Date` and DOM `Element`s can be subclassed.

Object construction for a function named `Ctor` now uses two-phases (both virtually dispatched):
- Call `Ctor[@@create]` to allocate the object, installing any special behavior
- Invoke constructor on new instance to initialize

The known `@@create` symbol is available via `Symbol.create`.  Built-ins now expose their `@@create` explicitly.

```JavaScript
// Pseudo-code of Array
class Array {
    constructor(...args) { /* ... */ }
    static [Symbol.create]() {
        // Install special [[DefineOwnProperty]]
        // to magically update 'length'
    }
}

// User code of Array subclass
class MyArray extends Array {
    constructor(...args) { super(...args); }
}

// Two-phase 'new':
// 1) Call @@create to allocate object
// 2) Invoke constructor on new instance
var arr = new MyArray();
arr[1] = 12;
arr.length == 2
```

### Math + Number + String + Object APIs
Many new library additions, including core Math libraries, Array conversion helpers, and Object.assign for copying.

```JavaScript
Number.EPSILON
Number.isInteger(Infinity) // false
Number.isNaN("NaN") // false

Math.acosh(3) // 1.762747174039086
Math.hypot(3, 4) // 5
Math.imul(Math.pow(2, 32) - 1, Math.pow(2, 32) - 2) // 2

"abcde".includes("cd") // true
"abc".repeat(3) // "abcabcabc"

Array.from(document.querySelectorAll('*')) // Returns a real Array
Array.of(1, 2, 3) // Similar to new Array(...), but without special one-arg behavior
[0, 0, 0].fill(7, 1) // [0,7,7]
[1, 2, 3].find(x => x == 3) // 3
[1, 2, 3].findIndex(x => x == 2) // 1
["a", "b", "c"].entries() // iterator [0, "a"], [1,"b"], [2,"c"]
["a", "b", "c"].keys() // iterator 0, 1, 2
["a", "b", "c"].values() // iterator "a", "b", "c"

Object.assign(Point, { origin: new Point(0,0) })
```

### Binary and Octal Literals
Two new numeric literal forms are added for binary (`b`) and octal (`o`).

```JavaScript
0b111110111 === 503 // true
0o767 === 503 // true
```

### Promises
Promises are a library for asynchronous programming.  Promises are a first class representation of a value that may be made available in the future.  Promises are used in many existing JavaScript libraries.

```JavaScript
function timeout(duration = 0) {
    return new Promise((resolve, reject) => {
        setTimeout(resolve, duration);
    })
}

var p = timeout(1000).then(() => {
    return timeout(2000);
}).then(() => {
    throw new Error("hmm");
}).catch(err => {
    return Promise.all([timeout(100), timeout(200)]);
})
```

### Reflect API
Full reflection API exposing the runtime-level meta-operations on objects.  This is effectively the inverse of the Proxy API, and allows making calls corresponding to the same meta-operations as the proxy traps.  Especially useful for implementing proxies.

```JavaScript
// No sample yet
```

### Tail Calls
Calls in tail-position are guaranteed to not grow the stack unboundedly.  Makes recursive algorithms safe in the face of unbounded inputs.

```JavaScript
function factorial(n, acc = 1) {
    'use strict';
    if (n <= 1) return acc;
    return factorial(n - 1, n * acc);
}

// Stack overflow in most implementations today,
// but safe on arbitrary inputs in ES6
factorial(100000)
```
