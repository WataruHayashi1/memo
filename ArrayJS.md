# 非破壊的な配列操作 : `for-of` vs `.reduce()` vs `.flatMap()`

[Processing Arrays non-destructively](https://2ality.com/2022/05/processing-arrays-non-destructively.html)の日本語訳

ここでは、以下の3つの配列操作について注目する。

- `for-of`ループ
- `.reduce()`メソッド
- `.flatMap()`メソッド

本文では、配列操作をする際にそれぞれのメソッドの特徴からどれを選ぶかの判断を手助けすることを目標とする。
`.reduce()`や`.flatMap()`について知らなくても、ここで説明するので心配することはない。

それぞれの特徴について理解を深めるために、次の機能を実装する。

- 配列をフィルターし出力する
- 各配列要素をマッピングし、1つの要素として出力する
- 各配列要素を0個以上の要素に展開する
- フィルターマッピング(フィルターとマッピングを一度に実行する)
- 配列の統計量を計算する
- 配列要素を見つける
- 配列の状態を調べる

これら全てを非破壊的(入力の配列が変わらないこと)に実装する。出力される配列は、元の配列を操作したものでなく新しく生成されたものである。

- [非破壊的な配列操作 : `for-of` vs `.reduce()` vs `.flatMap()`](#非破壊的な配列操作--for-of-vs-reduce-vs-flatmap)
  - [1 `for-of`ループの配列操作](#1-for-ofループの配列操作)
    - [1.1 `for-of`でフィルタリング](#11-for-ofでフィルタリング)
    - [1.2 `for-of`でマッピング](#12-for-ofでマッピング)
    - [1.3 `for-of`で展開](#13-for-ofで展開)
    - [1.4 `for-of`でフィルターマッピング](#14-for-ofでフィルターマッピング)
    - [1.5 `for-of`で統計量を計算](#15-for-ofで統計量を計算)
    - [1.6 `for-of`で見つける](#16-for-ofで見つける)
    - [1.7 `for-of`で状態を調べる](#17-for-ofで状態を調べる)
    - [1.8 `for-of`を使う](#18-for-ofを使う)
    - [1.9 ジェネレーターと`for-of`](#19-ジェネレーターとfor-of)
  - [`.reduce()`メソッド](#reduceメソッド)
    - [2.1 `.reduce()`でフィルタリング](#21-reduceでフィルタリング)
    - [2.2 `reduce()`でマッピング](#22-reduceでマッピング)
    - [2.3 `.reduce()`で展開](#23-reduceで展開)
    - [2.4 `.reduce()`でフィルターマッピング](#24-reduceでフィルターマッピング)

## 1 `for-of`ループの配列操作

`for-of`は非破壊的に配列変形を行うことができる。

- 変数`result`と空の配列を宣言する
- 入力された配列のそれぞれの要素`elem`が
  - 値が`result`に追加され
    - 必要に応じて`elem`を変形し、`result`に追加する

### 1.1 `for-of`でフィルタリング

`for-of`を使って配列操作の感覚をつかみ、(シンプルな)`.filter()`メソッドを実装してみる。

```js
function filterArray(arr, callbacks) {
  const result = [];
  for (const elem of arr) {
    if (callback(elem)) {
      result.push(elem);
    }
  }
  return result;
}
 
assert.deepEqual (
  filterArray(['', 'a', '', 'b'], str => str.length > 0),
  ['a', 'b`]
)
```

### 1.2 `for-of`でマッピング

`for-of`を使って`.map()`メソッドも実装する。

```js
function mapArray(arr, callback) {
  const result = [];
  for (const elem of arr) {
    result.push(callback(elem));
  }
  return result;
}

assert.deepEqual (
  mapArray(['a', 'b', 'c'], str => str + str),
  ['aa', 'bb', 'cc']
);
```

### 1.3 `for-of`で展開

`collectFruits()`は配列中の人が持っている全ての果物を返す

```js
function collectFruits(persons) {
  const result = [];
  for (const person of persons) {
    result.push(...person.fruits);
  }
  return result;
}

const PERSONS = [
  {
    name: 'Jane',
    fruits: ['strawberry', 'raspberry'],
  },
  {
    name: 'John',
    fruits: ['apple', 'banana', 'orange'],
  },
  {
    name: 'Rex',
    fruits: ['melon'],
  },
];

assert.deepEqual (
  collectFruits(PERSONS),
  ['strawberry', 'raspberry', 'apple', 'banana', 'orange', 'melon']
);
```

### 1.4 `for-of`でフィルターマッピング

```js
/**
 * 評価が'minRating'以上の映画のタイトル
 */
function getTitles (movies, minRating) {
  const result = [];
  for (const movie of movies) {
    if (movie.rating >= minRating) { // (A)
      result.push(movie.title); // (B)
    }
  }
  return result;
}

const MOVIES = [
  { title: 'Inception', rating: 8.8 },
  { title: 'Arrival', rating: 7.9 },
  { title: 'Groundhog Day', rating: 8.1 },
  { title: 'Back to the Future', rating: 8.5 },
  { title: 'Being John Malkovich', rating: 7.8 },
];

assert.deepEqual (
  getTitles(MOVIES, 8),
  ['Inception', 'Groundhog Day', 'Back to the Future']
);
```

- フィルタリングは、A行の`if`文とB行の`.push()`メソッドで行われている
- `movie.title`をプッシュすることでマッピングされている(`movie`要素を追加してはいけない)

### 1.5 `for-of`で統計量を計算

`getAverageGrade()`は配列中の生徒の成績の平均を計算する

```js
function getAverageGrade (students) {
  let sunOfGrades = 0;
  for (const student of students) {
    sumOfGrades += student.grade;
  }
  return cumOfGrades / students.length;
}

const STUDENTS = [
  {
    id: 'qk4k4yif4a',
    grade: 4.0,
  },
  {
    id: 'r6vczv0ds3',
    grade: 0.25,
  },
  {
    id: '9s53dn6pbk',
    grade: 1,
  },
];

assert.equal (
  getAverageGrade(STUDENTS),
  1,75
);
```

注意 : デシマルの端数の計算結果は丸め誤差が発生する([詳細](https://exploringjs.com/impatient-js/ch_numbers.html#the-precision-of-numbers-careful-with-decimal-fractions))

### 1.6 `for-of`で見つける

`for-of`はソートされてない配列中の特定の要素も探索できる

```js
function findInArray (arr, callback) {
  for (const [index, value] of arr.entries()) {
    if (callback(value)) {
      return {index, value}; // (A)
    }
  }
  return undefined;
}

assert.deepEqual (
  findInArray(['', 'a', '', 'b'], str => str.length > 0),
  {index: 1, value: 'a'}
);
assert.deepEqual (
  findInArray(['', 'a', '', 'b'], atr => str.length > 1),
  undefined
);
```

ここでは、目的のものを1度でも見つけたときに、`return`で早々にループを抜けることに意義がある(A行)

### 1.7 `for-of`で状態を調べる

配列に対して`.every()`メソッドを使うと、ループを早々に終了することができる(A行)

```js
function everyArrayElement (arr, condition) {
  for (const elem of arr) {
    if (!condition(elem)) {
      return false; // (A)
    }
  }
  return true;
}

assert.equal (
  everyArrayElement(['a', '', 'b'], str => str.length > 0),
  false
);
assert.equal (
  everyArrayElement(['a', 'b'], str => str.length > 0),
  true
);
```

### 1.8 `for-of`を使う

`for-of`は配列操作において非常に多機能なツールである :

- 直感的な挿入操作で配列を作ることができる
- 結果が配列でないとき、`return`や`break`でループをはやく終わらせやすい

他の`for-of`の利点は、

- [synchronous iterables](https://exploringjs.com/impatient-js/ch_sync-iteration.html)で動作する。加えて、[`for-await-of`ループ](https://exploringjs.com/impatient-js/ch_async-iteration.html#for-await-of)に切り替えることにより、[asynchronous iterables](https://exploringjs.com/impatient-js/ch_async-iteration.html)もサポートできる
- 使用が許される場合、`await`と`yield`を関数内で使うことができる

`for-of`の欠点は、解決使用とする問題によっては他の解法より冗長になりうることである

### 1.9 ジェネレーターと`for-of`

`yield`については前節で触れているが、加えて指摘したい点がある。それは、ストリームアイテムのオンデマンド処理によるストリームからわかるように、ジェネレータが同期・非同期の反復処理の生成・実行においてどれだけ便利なものであるかということである。

次の例のように、同期ジェネレータで`.filter()`と`.map()`を実装する

```js
function* filterIterable(iterable, callback) {
  for (const item of iterable) {
    if (callback(item)) {
      yield item;
    }
  }
}

const iterable1 = filterIterable (
  ['', 'a', '', 'b'],
  str => str.length > 0
);

assert.deepEqual (
  Array.from(iterable1),
  ['a', 'b']
);

function* mapIterable (iterable, callback) {
  for (const item of iterable) {
    yield callback(item);
  }
}

const iterable2 = mapIterable(['a', 'b', 'c'], str => str + str);
assert.deepEqual (
  Array.from(iterable2),
  ['aa', 'bb', 'cc']
);
```

## `.reduce()`メソッド

`.reduce()`メソッドは配列の統計量を計算するのに用いられる。`.reduce()`は以下のアルゴリズムに基づいている。

- 空の配列に対して有効な統計量を初期化する
- 配列をループし、それぞれの要素ごとに
  - 現在の要素と前の統計量を組み合わせた新しい統計量を計算する

`.reduce()`が登場する前は、`for-of`を使ってアルゴリズムを実装していた。以下に、例として配列の連結を挙げる

```js
function concatElements(arr) {
  let summary = ''; // 初期化
  for (const element of arr) {
    summary = summary + element; // 更新
  }
  return summary;
}

assert.equal (
  concatElements(['a', 'b', 'c']),
  'abc'
);
```

`.reduce()`メソッドはループと合計値の追跡をするので、ユーザは初期化と更新に注目できる。以後は、合計値の大まかな言い換えとしてアキュムレータという単語を使う。`.reduce()`は2つのパラメータを持つ。

1. コールバック
  - 入力:前のアキュムレータと現在の要素
  - 出力:新たなアキュムレータ
2. アキュムレータの初期値

以下のコードでは、`concatElements()`の実装に`.reduce()`を使っている。

```js
const concatElements = (arr) => arr.reduce (
  (accumulator, element) => accumulator + element, // 更新
  '' // 初期値
);
```

### 2.1 `.reduce()`でフィルタリング

`.reduce()`は非常に汎用的である。以下ではフィルタリングを実装する。

```js
const filterArray = (arr, callback) => arr.reduce (
  (acc, elem) => callback(elem) ? [...acc, elem] : acc,
  []
);

assert.deepEqual (
  filterArray(['', 'a', '', 'b'], str => str.length > 0),
  ['a', 'b']
);
```

残念なことに、JavaScriptの配列は非破壊的に要素を追加することに関しては、あまり効率的でない(他の関数型言語の連結リストとは対照的に)。したがって、以下のようにアキュムレータを変化させるほうが効率的である。

```js
const filterArray = (arr, callback) => arr.reduce (
  (acc, elem) => {
    if (callback(elem)) {
      acc.push(elem);
    }
    return acc;
  },
  []
);
```

### 2.2 `reduce()`でマッピング

```js
const mapArray = (arr, callback) => arr,reduce (
  (acc, elem) => [...acc, callback(elem)],
  []
);

assert.deepEqual (
  mapArray(['a', 'b', 'c'], atr => str + str),
  ['aa', 'bb', 'cc']
);
```

より効率的なバージョン

```js
const mapArray = (arr, callback) => arr.reduce (
  (acc, elem) => {
    acc.push(callback(elem));
    return acc;
  },
  []
);
```

### 2.3 `.reduce()`で展開

```js
const collectFruits = (persons) => persons.reduce (
  (acc, person) => [...acc, ...person.fruits],
  []
);

const PERSONS = [
  {
    name: 'Jane',
    fruits: ['strawberry', 'raspberry'],
  },
  {
    name: 'John',
    fruits: ['apple', 'banana', 'orange'],
  },
  {
    name: 'Rex',
    fruits: ['melon'],
  },
];

assert.deepEqual (
  collectFruits(PERSONS),
  ['strawberry', 'raspberry', 'apple', 'banana', 'orange', 'melon']
);
```

別バージョン

```js
const collectFruits = (persons) => persons.reduce(
  (acc, person) => {
    acc.push(...person.fruits);
    return acc;
  },
  []
);
```

### 2.4 `.reduce()`でフィルターマッピング

```js
const getTitles = (movies, minRating) => movies.reduce (
  (acc, movie) => (movie.rating >= minRating)
    ? [...acc, movie.title]
    : acc,
  []
);

const MOVIES = [
  { title: 'Inception', rating: 8.8 },
  { title: 'Arrival', rating: 7.9 },
  { title: 'Groundhog Day', rating: 8.1 },
  { title: 'Back to the Future', rating: 8.5 },
  { title: 'Being John Malkovich', rating: 7.8 },
];

assert.deepEqual (
  getTitles(MOVIES, 8),
  ['Inception', 'Groundhog Day', 'Back to the Future']
);
```

より効率的なバージョン

```js
const getTitles = (movies, minRating) => movies,reduce (
  (acc, movie) => {
    if (movie.rating >= minRating) {
      acc.push(movie.title);
    }
    return acc;
  },
  []
);
```