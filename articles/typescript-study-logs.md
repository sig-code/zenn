---
title: "TypeScript勉強会の復習"
emoji: "👣"
type: "tech"
topics: ["typescript"]
published: true
---
# TypeScript勉強会の復習

先日、私が参加している勉強会で、一問一答のTypeScriptで復習する機会がありました。
車輪の再発明ですが、自分の理解を深めるために、自分なりの言葉でまとめました。

## typeとinterfaceの違い

### interface
- 宣言を重複することができる→オブジェクトを追加することが可能
- プロパティの重複は許可されない

```tsx
interface Window {
  id: number
  title: number
}

interface Window  {
  title: string
}
// Subsequent property declarations must have the same type.  Property 'title' must be of type 'number', but here has type 'string'
```

**interfaceの拡張**

```tsx
interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}
```

### type
- 一度宣言したら使用することができない

```tsx
type Animal = {
    name: string
}

type Animal = {
    id: number
}
// Duplicate identifier 'Animal'.
```

どちらかに優劣があるわけではない。

- メンバーの話だと、現場によって、運用方法はさまざま
- ルール作りをして、どちらを利用するかを決定するのもよいかも

プロパティの上書きをする時に挙動が異なる

interfaceは先述した通り。

typeでプロパティの上書きを使用する時↓

```tsx
type Animal = {
    id: number
    name: string
}

type Bear = Animal & {
    id: string
    hobby: string
}
//　この時点ではエラーが出ない

const human : Bear = {
    id: 1, // → Type 'number' is not assignable to type 'never'.
    name: 'taro',
    hobby: 'programming'
}

// 重複しているidはnever型となっている
```

## primitive型とは
- 基本的な型で、言語に組み込まれている型
- プリミティブ型は7種類 `boolean`, `number`, `string`, `undefined`, `null`, `symbol`, `bigint`
- イミュータブル特性・プロパティを持たない（[サバイバルTypeScript](https://typescriptbook.jp/reference/values-types-variables/primitive-types)より）

## is in の違い

## is
`is`は`TypeScript`の型推論を補強するもの。narrowingの時に使用する

使用例（[こちらの記事](https://qiita.com/ryo2132/items/ce9e13899e45dcfaff9b)より）

```tsx
const isString = (test: unknown): boolean => {
	return typeof test === "string";
}; // string型への絞り込みをする関数

const example = (foo: unknown) => {
	if (isString(foo)) {
		console.log(foo.length); // Error
	}
}
// 上記ではError箇所で fooはまだunknownとして型推論される
// なぜなら、isString関数スコープないで、型の絞り込みが行われて完結してる
// →isString関数がtrueな場合でもfooがunknown型として推論されてしまう
```

上記の様な場合は `is`の出番
```tsx
const isString = (test: unknown): test is string => {
	return typeof test === "string";
}; // isを使用するとtrueが返って来た時に、引数の型がstring型であるとコンパイラに教えることができる

const example = (foo: unknown) => {
	if (isString(foo)) {
		console.log(foo.length) // fooはstringとして推論される
	}
}
```

`is`はオブジェクト型の型絞り込みにも使用可能。
使用例↓
```tsx
type Bird = {
  fly : () => {
    // Do something
  };
};

type Fish = {
  swim: () => {
    // Do something
  };
};

const example = (fishOrBird: Fish | Bird) => {
  if ((fishOrBird as Bird).fly() !== undefined) {
    console.log(fishOrBird.fly);
  } else {
    console.log(fishOrBird.swim);
  }
};

// 上記のfishOrBird.fly,fishOrBird.swimでは Fish | Bird 型として型推論
// →type errorが発生する
// →型を絞り込めていない
```

上記のような時に`is`で型を指定することができる

```tsx
const isBird = (test: Fish | Bird): test is Bird => {
  return (test as Bird).fly() !== undefined;
}; // trueが返ればBird falseが返ればFishに

const example = (fishOrBird: Fish | Bird) => {
  if (isBird(fishOrBird)) {
    console.log(fishOrBird.fly); // Bird型に
  } else {
    console.log(fishOrBird.swim); // Fish型に
  }
}
```

注意点は元記事を参考にしてください


## in
`in`は2つの意味を持つ構文
- JSにもあるオブジェクトが特定のプロパティを持つか判定→型の絞り込みに使用可能
- `keyof`構文と組み合わせて`mapped type`の定義に使用可能

使用例↓

**型の絞り込み**
`in`を使えば、先述の判定処理を関数に切り出し`is`で[type predicate](https://qiita.com/kgtkr/items/7e4f18224c3362ceceeb)を記述する必要もありません。
```tsx
const example = (fishOrBird: Fish | Bird) => {
  // fishOrBirdが'fly'というプロパティを持っているかどうか
  if (fly in fishOrBird) console.log(fishOrBird.fly); // Bird型
  console.log(fishOrBird.swim) // Fish型
};
```

**mapped typeでの利用**
`keyof`構文と組み合わせて`mapped type`で新しい方を定義できます。

```tsx
type Dog = {
  name: string;
  run: () => void;
};

type PartialDog = {
  [P in keyof Dog]?: Dog[P];
}
// keyofを使用し、オブジェクトのキーをユニオン型に変更
// inで反復処理して、それぞれのプロパティにオプションプロパティ(optional property)を付与する
// みたいなイメージ
```

## オプションプロパティ・オプショナルチェーン
オプションプロパティの使用例↓（プロパティ名の後ろに`?`を書く）
```tsx
let human: {
  height: number,
  weight?: number,
  bmi?: number,
  };
```

オプションプロパティを持ったオブジェクト型には、そのオプションプロパティを持たないオブジェクトを代入できる
```tsx
human = {
  height: 170 //OK
};
```
また、オプションプロパティの値が`undefined`でも代入可能
しかし、オプションプロパティの値が`null`の場合は代入できない
```tsx
human = {
  height: 170,
  weight: null; // Type 'null' is not assignable to type 'number | undefined'.
} // →オプションプロパティは' <T> | undefinedとなるていうことかも
```

オプショナルチェイニング演算子
JavaScriptのオプショナルチェーン`?.`はオブジェクトのプロパティが存在しない時でも、エラーを起こさずにプロパティを参照できる

JavaScriptでは`null`や`undefined`のプロパティを参照するとエラーに
```JSX
const book = undefined;
const title = book.title;
// TypeError: Cannot read property 'title' of undefined
```

オプショナルチェーンを使用すると
```jsx
const book = undefined;
const title = book?.title;

console.log(title);
// →undefined
```

オプショナルチェーンは色んな場面で使えそう

その他参考↓
- [https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Optional_chaining](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Optional_chaining)
- [https://typescriptbook.jp/reference/values-types-variables/object/optional-chaining](https://typescriptbook.jp/reference/values-types-variables/object/optional-chaining)
- [https://typescriptbook.jp/reference/values-types-variables/object/optional-property](https://typescriptbook.jp/reference/values-types-variables/object/optional-property)

## 非nullアサーション演算子（!）
- 型推論ができない文脈において、値が`非null`,`undefined`ではないことを主張する。
- ただし、値が`null`や`undefined`ではないことわかっている場合のみ使用するべき
- null許容型`(T | null | undefined)`に対して、使用すると`null | undefined`ではなく`T`であることをコンパイラに明示できる

```tsx
document.get
```

- [https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#non-null-assertion-operator-postfix-](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#non-null-assertion-operator-postfix-)
- [https://zenn.dev/oreo2990/articles/3d780560c5e552#%E9%9D%9Enull%E3%82%A2%E3%82%B5%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E6%BC%94%E7%AE%97%E5%AD%90(!)](https://zenn.dev/oreo2990/articles/3d780560c5e552#%E9%9D%9Enull%E3%82%A2%E3%82%B5%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E6%BC%94%E7%AE%97%E5%AD%90(!))

## Null合体演算子（??）
左の値が`null`または`undefined`のときに右の値を返します。そうでない場合は左の値を返す

```tsx
console.log(undefined ?? 1)
// → 1
console.log(2 ?? 1);
// → 2

// 意図しない挙動かも
console.log('' ?? 'ゲスト')
// → ''

// 上記のときは論理和を使うとよさそう
console.log('' || 'ゲスト')
// → 'ゲスト'
```

この時、falsyな値の時に右の値を返すわけではないことに注意しないといけない。
空文字などで右の値を返してほしい場合は`Null合体演算子`を使用するのではなく、`||(論理和)`を使用する方が良いと思われる

## void型とnever型の違い
結論：void型は**何も返さない**を表現する型、never型は**けっして起こりえないこと**を表現する型
- void型→正常終了時に何も値を返さないメソッド
- never型→正常終了せずなにも返ってこないメソッド（Errorをthrowするなど）

never型は状態管理のスイッチ文でのユースケース
```tsx
type Signal = 'green' | 'red' | 'hoge';

const getMessageFromSignal = (color: color): string => {
  switch (color) {
    case 'green': {
      return 'Go!';
    }
    case 'red': {
      return 'Stop!';
    }
    default: {
      const strangeValue: never = color; // Type 'string' is not assignable to type 'never'.
      throw new Error(`${strangeValue} is not color.`);
    }
  }
};
// このように後からアクションを変更する時に、エラーを吐いてくれるようにすることができる。
// voidでも良いが、関数型の値が入ってくる時に弾くことができないので、neverの方がいいのではという意見があった。
```

## undefined null any unknownの違い
### まずundefinedとnullの違い
undefinedとnullは**値がない**ことを意味するが、
- undefined→値が代入されていないため、値がない
- null→代入すべき値が存在しないため、値がない

また、**nullは自然発生しない**。
`undefined`は言語仕様上、明示しなくても自然に発生するが、`null`はプログラマーが意図的に使用しない限り発生しない

### any unknownの違い
anyとunknown型はどのような値でも代入できる
違いとしては、any型に代入したオブジェクトのプロパティ、メソッドは使用することができる
```tsx
const a: any = 'hoge'
console.log(a.toFixed()) //実行時にはエラーになるが、anyだと使用できる

const a: unknown = 'hoge'
console.log(a.toFixed()) // Object is of type 'unknown',
```

unknown型は一貫して、TypeScriptがプロパティ、メソッドへのアクセスを行わせない

## keyof typeofの違い
### keyof
`keyof`はオブジェクト型からプロパティ名を型として返す演算子

```tsx
type Person = {
  name: string
};

type PersonKey = keyof Person;
// 上は次と同じ意味
type PersonKey = "name";
```

2つ以上のプロパティがあるオブジェクト型に`keyof`を使用した場合はプロパティ名がユニオン型で返される
```tsx
type Book = {
  title: string;
  price: number;
  rating: number;
};
type BookKey = keyof Book;
// 上は次と同じ意味になる
type BookKey = "title" | "price" | "rating";

```

mapped typeに`keyof`を用いると、そのキーの型が返ります。
```tsx
type MapLike = { [K in "x" | "y" | "z"]: any };
type MapKeys = keyof MapLike;
//=> "x" | "y" | "z"
```

その他にもおもしろい挙動があったので、参考記事を見てみてください。
参考: [https://typescriptbook.jp/reference/type-reuse/keyof-type-operator](https://typescriptbook.jp/reference/type-reuse/keyof-type-operator)

### typeof
値の型を調べることができる

:::message
typeof nullの演算結果はnullではなく、objectなので注意
:::

## 可変長引数(rest parameter)
引数の数に決まりがない関数を作成することができる。引数の個数が決まっていない引数のことを可変長引数という。JavaScriptでは可変長引数は残余引数と呼ぶらしい

こんな感じ書く。
```tsx
function func(...params) {
  // ...
}
```

普通の引数と組み合わせる時↓
```tsx
function func(age: number, ...params: number[]) {
  console.log(age, params)
}

func(1,2,3)
//→ 1, [2, 3]
```
:::message
- 残余引数は必ず最後の引数でないといけない
- 残余引数を複数持たせることはできない
:::

スプレッド構文は配列を引数にバラすものです


## Conditional Typesおよびinfer
`Conditional`とは条件、条件付きのという意味ですので、`Conditional Types`とは、型の条件分岐です。

例
```tsx
type MyCondition<T, U, X, Y> = T extends U ? X : Y;
```
三項演算子と同様、TがUに代入可能であればX。そうでなければYという型を表しています。

Conditional Typesの性質
- 遅延評価:X,Yの決定に対して、T,Uという型変数への依存がある場合、型の解決はT,Uが決定されるまで評価が遅延される
- Union typesの分配則: Union typesのConditional Typesは、それぞれのConditional TypesのUnionに展開される
例
```tsx
(T1 | T2) extends U ? X : Y = (T1 extends U ? X : Y) | (T2 extends U ? X : Y)
```


```tsx
type Diff<T, U> = T extends U ? never : T;
type RequiredKeys = Diff<"age" | "name", "age">; // "name"

// Diff関数はUがTの部分型であれば、neverを返す関数
//2行目ではage型が該当するので、ageがnever型になる→never | nameのUnionになる→name型となる？
```

Conditional Typesが型における、条件マッチングを可能に→条件分岐中に保存した型を再利用することができる
→Type inference in Conditional Types

Conditional Typesの`T extends U ? X : Y`の条件（Uのとこ）に`infer S`とかくと、Sに補足された型をXの部分で再利用可能になる

具体例↓
```tsx
//Utility TypesのReturnTypeの実装
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
```

`(...args: any[]) => any`だと引数もなんでもよい、返り値もなんでもよいなので、「関数なんでも」になる。
この返り値部分を`infer R`と書き換えたものが、マッチング対象の型である`(...args: any[]) => infer R`です。
`infer R`が「マッチした時、その部分に推論される型をRにいれる（保存する）」という意味になる
つまり、`ReturnType<T>`という型は「Tが関数であればその関数の返り値の型」を表すことになる

例:
```tsx
function bmi(height: number, weight: number) {
    const bmi = weight / height**2
    if(bmi > 20) return 'ぽっちゃり'
    return 'やせているか普通'
}

type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
type hoge = ReturnType<typeof bmi> // →戻り値で'ぽっちゃり'と'やせているか普通'のユニオン型になる→hogeがユニオン型に
const a : hoge = bmi(1.7, 20)
console.log(a)
```

以下はメモ程度です↓
まず、ここまでの情報で参考にした記事
[TypeScriptの型入門](https://qiita.com/uhyo/items/e2fdef2d3236b9bfe74a)
[TypeScriptの型初級](https://qiita.com/uhyo/items/da21e2b3c10c8a03952f)
[TypeScriptの'infer'を一撃で理解する](https://reosablo.hatenablog.jp/entry/2020/08/25/005957)
[TypeScript2.8のConditional Typesについて](https://qiita.com/Quramy/items/b45711789605ef9f96de)



私はここまで遡らないとだめでした。。。
- Conditional Types
- Mapped Types
- したのコードを読み解く

```tsx
interface MyObj {
  foo: string;
  bar: number;
}

// strの型はstringとなる
const str: MyObj['foo'] = '123';
```
この例では`MyObj['foo']`という型が登場しています。上で見た`T[K]`という構文と比べると、`T`が`MyObj`型で`K`が`'foo'`型となります。

よって、`MyObj['foo']`は`MyObj`型のオブジェクトの`foo`というプロパティの型であるstringとなります。

`T[K]`という構文は`K`が`keyof T`の部分型である必要がある。

`K extends keyof T`→宣言する型変数`K`は`keyof T`の部分型でなければならないという意味
`{[P in K]: T}`→`K`型の値として可能な各文字列`P`に対して、型`T`を持つプロパティが存在するようなオブジェクトの型
すなわち、`{[P in 'foo' | 'bar']: number}` というのは`{ foo: number; bar: number; }`と同じ意味です

例↓
```tsx
type Id<T> = T extends { id: infer U } ? U : never;
// Tがidというプロパティを持っている場合 → idの型を返す
// idというプロパティがなければneverを返す
```


おまけ
#### Widening Literal TypesとNonWidening Literal Types
**Widening Literal Types**
```tsx
const widening = 'HOGE'; // "HOGE"型になっている
const test = {
  widening // string型になってしまう！
}
test.widening = '文字列' // string型なので代入可能
// wideningを定義した時は"HOGE"のみ許容するString Literal Typesなのですが、オブジェクトのプロパティとすると純粋なString Literal Typesになってしまう
//→これがWidening Literal Typesの挙動
```

上記のようなオブジェクトの中でも`HOGE`のみを許容する`String Literal Types`を定義したい時は`as const`を使用し、`NonWidening Literal Types`として定義することで挙動を変更することができる。

例↓
```tsx
const nonWidening = 'HOGE' as const // HOGE型
const test = {
  nonWidening // ここでもHOGE型
};
test.nonWidening = '文字列' // Error: Type '文字列' is not assignable type 'HOGE'
```

`const アサーション`は配列への再代入ができなくなっている。
色々と使える場面がありそう。
