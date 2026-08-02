---
title: "JavaGold SE17 学習メモ"
emoji: "☕"
type: "tech"
topics: ["javagold"]
published: true
slug: memo_javagold-se17
---

## １章　コレクションとジェネリクス


◆型パラメータ  

型パラメータが使用できない場所
- 型パラメータを使用したインスタンス化。具体的な型がわかってないため  
`private E e = new E();` ⇒×
- staticつきからの使用（インスタンス生成されて具体的な型が決まるため？）。  
ただしジェネリックメソッドはstaticにできる！  
`private static E e2;`⇒×  
`public static E errMethod(E e3) { return e3 };`⇒ジェネリックメソッドとして定義してないのでNG  
`static <E> void bar(List<E> list) { };`⇒これはOK  

OKなケース

- 配列は参照型なので、\<int[]>とかはOK。

・ワイルドカードについて  

**！！重要！！**  
ワイルドカードは新しい型を作り出すわけではない。すでに存在する型を汎用的に受け取るためのもの。なので、ジェネリック定義側に?は使えない！  
`public class Human<? extends Number> { }` ←結局何型か分からないからここには使えない！ジェネリックメソッドも同じ。

**！！重要！！**  
`super`は<u>ワイルドカード付きでしか使えない</u>。そしてワイルドカードはジェネリック定義側には使えない。つまり`<T super Number>`これはNG。Numberの親クラスがどんな動作を持ってるかわからないため。

例：  
`Box<? super Number> box;`  
⇒なんの型かは分からないけど、Box\<Object>もBox\<Number>も参照できる型。Box\<T>の実際のTは実体による。

変数宣言時の型にワイルドカード使用可能。ただし  
インスタンス生成側は具体的な型を指定する必要あるので使用不可！  
`Queue<? super Number> queue = new ArrayDeque<? super Number>();` ⇒NG  
`Queue<? super Number> queue = new ArrayDeque<>();` ⇒これはOK  

**PECS (Producer Extends, Consumer Super)** の考え方 

|宣言|追加できるもの|取り出せる型|
|--|--|--|
|非境界：<?>|null以外追加できない|Object|
|上限境界：<? extends T>|null以外追加できない|T|
|下限境界：<? super T>|Tとそのサブクラス|Object|
||||

詳細： 
- `<? extends Number>`で宣言した場合  
・実際の中身がIntegerなのかDoubleなのかその他サブクラスなのかコンパイル時点で判断できないので型安全のために追加を許可しない。  
・実際の中身がNumber以下であることは確定しているため、取り出しはNumberで受けられる。
```java
List<? extends Number> list = new ArrayList<Integer>();
list.add(10); list.add(11L); list.add(22.1F); //←addした時点でコンパイルエラー
```

- `<? super Number>`で宣言した場合  
・実際の中身がNumber以上であることが確定しているため、Number以下の追加は可能。  
・実際の中身がNumberなのかObjectなのか判断できないので、<u>取り出しはObjectでのみ受けられる</u>。  
```java
List<? super Number> list = new ArrayList<Object>();
list.add(10); list.add(11L); list.add(22.1F); //←addした時点でコンパイルエラー
```


◆ジェネリックメソッド  
・`public static <E> void bar() { }`の順番（ジェネリクスは戻り値の前）。  
・呼び出し側での型指定方法は、  
`obj.<Integer>foo();`や`List<Integer> list1 = obj.foo();`（obj.<>foo()はダメ）でOK。

◆コレクション

---

```

Iterable <<interface>>
└──Collection <<interface>>
　　├── List <<interface>>
　　│   ├── ArrayList ●
　　│   ├── LinkedList ●
　　│   └── Vector ●
　　│
　　├── Set <<interface>>
　　│   ├── HashSet ●
　　│   │   └── LinkedHashSet ●
　　│   └── SortedSet <<interface>>
　　│       └── NavigableSet <<interface>>
　　│           └── TreeSet ●
　　│
　　└── Queue <<interface>>
　　    └── Deque <<interface>>
　　        ├── ArrayDeque ●
　　        └── LinkedList ●

　　　　Map <<interface>>
　　　　├── HashMap ●
　　　　│   └── LinkedHashMap ●
　　　　├── SortedMap <<interface>>
　　　　│   └── NavigableMap <<interface>>
　　　　│       └── TreeMap ●
　　　　├── Hashtable ●
　　　　└── ConcurrentHashMap ●

```

---

| クラス・インターフェース | メソッド | メモ |
| --- | --- | --- |
| Arrays | List\<E> asList(…) | (Collectionsクラスには無い！)。戻り値の型はArrayListではなくList！！ |
| List\<E> | List\<E> subList(int, int) | 切り出してリスト作成。元のリストが変更不可(List.of生成とか)だと切り出したリストも変更不可。subSetやsubMapもあるぽい |
| Deque\<E> | void addFirst, addLast | 追加。失敗時はIllegalStateException |
| Deque\<E> | boolean offerFirst, offerLast | 追加。失敗時はfalse |
| Deque\<E> | E removeFirst, removeLast | 削除。失敗時はNoSuchElementException |
| Deque\<E> | E pollFirst, pollLast | 削除。失敗時はnull |
| Deque\<E> | E getFirst, getLast | 取得。失敗時はNoSuchElementException |
| Queue\<E> | E element | 取得。失敗時はNoSuchElementException |
| Deque\<E> | E peekFirst, peekLast | 取得。失敗時はnull |
| Map<K, V> | static entry(key, value) | Entryペア１個分作る。**作ったEntryは変更不可** |
| Map<K, V> | static ofEntries(…) | 引数のEntryペアからMapを作る。**作ったMapは変更不可** |
|  |  |
|  |  |

Dequeに対して両端キューとスタック（後入れ先出し）の操作可能。
```java
Deque<String> stack = new ArrayDeque<>();
stack.push("First");
stack.push("Second");
stack.add("A");
stack.addFirst("B");
stack.addLast("C");
stack.addLast("D");
System.out.println("-- Stack -- " + stack); // -- Stack -- [B, Second, First, A, C, D]
```

Setは重複NG。nullはHashSet、LinkedHashSetでは1個だけ入る。TreeSetはだめ（例外発生）。  
ArrayDequeクラスもnull要素禁止。追加(offerとかadd)しようとするとNullPointer発生。  

Tree～の要素はComparableインタフェースの実装必須。してないとClassCastException発生。

of()で作ったコレクションはいかなる変更も不可。  
Arrays.asList()で作ったリストは固定サイズの配列。つまり要素変更は可能、サイズ変更は不可。

MapはIterableを実装していないので、独自のforEach(**BiConsumer**)を持つ。  
もしくはMapのentrySetのforEach使えばOK。

## ２章　関数型インタフェースとラムダ式

### ネストクラス

◆メンバクラス

・非staticなネストクラス  
　外側クラスのインスタンスからインスタンス生成する  
　`Outer outer = new Outer();`  
　`Outer.Inner i1 = outer.`<u>`new Inner();`</u>

・staticなネストクラス  
　外側クラスのインスタンス不要でインスタンス生成できる
　  
　イメージ）  
　　外側クラスの中にある普通のクラス（ただし名前だけ所属してる）  
　　Outerというフォルダの中に Nested.java がある感じ  
　`Outer.Nested n1 = new `<u>`Outer.Nested();`</u>

◆ローカルクラス  
メソッドの中とかに定義するクラス。  
シールクラス宣言や、シールクラス指定先は不可。
修飾子としてabstract／final指定可能。  
外側ブロックのfinalまたは<u>実質的final※</u>なローカル変数にアクセス可能。  
（※変数の初期化以降に値が変更されていない変数）

◆無名クラス
一時的に継承したり実装したりするために使える。extendsやimplementsの明示的な記述は不可。  
シールクラス宣言や、シールクラス指定先は不可。
修飾子としてabstract／final指定不可。  
また、名前がないので明示的なコンストラクタ定義も不可。

### 関数型インタフェース

■メソッド参照  
関数型インタフェースの抽象メソッドと、「引数」と「戻り値」が互換性のある既存メソッドを、そのまま実装として使うときにメソッド参照が使える。  
クラス名::staticメソッド  
オブジェクト::インスタンスメソッド  
クラス名::インスタンスメソッド  
クラス名::new（コンストラクタ参照）  

**適用する関数型インタフェースの抽象メソッドの戻り値がvoidのときは、 戻り値がなんでも引数があってれば適用できる。戻り値は無視される！**

■java.util.functionパッケージの関数型インタフェース  
Consumerはaccept, Predicateはtest, Supplierはget, Functionはapply

■基本データ型の関数型インタフェース  
・Int, Long, Double | Consumer, Predicate, Supplier, Function  
の組み合わせ。SupplierだけはBooleanSupplierがある！  
戻り値が基本データ型の場合（Supplierと…To~Function）は、  
メソッド名が **基本のメソッド名As戻り値の型()** になる。その他は一緒。  
例：IntSupplierはgetAsInt, IntToDoubleFunctionはapplyAsDoubleなど。

■関数型インタフェースの合成（defaultメソッド）  
・Predicate系はand, or, negate  
　左から順に実行されるので優先順位はなし。  
・Function系はandThen, compose  
　andThenは左から右、composeはその逆（引数から実行）  

## ３章　JavaストリームAPI

Optionalのファクトリメソッドは３つ覚える（公開されたコンストラクタはないのでnewはできない）。  
値の取得系でOptional\<T>が返ってくるのはor(Supplier)のみ(メソッドチェーンみたいに使う用)。ほかは値Tそのまま返ってくる  

| メソッド | メモ |
| --- | --- |
| static empty() | 空のOptionalを返す |
| static of(T) | 生成。nullだとヌルポ |
| ofNullable(T) | 生成。nullだと空 |
|  |  |

java.util.streamパッケージ。

**コレクションのstream()はCollectionインタフェースのdefaultメソッドなのでMapは持っていない！**  
Mapで使うならkeySetとかentrySetなどしてからstream()する。

| メソッド | メモ |
| --- | --- |
| dropWhile(Predicate) | trueの間、スキップして捨てる |
| takeWhile(Predicate) | trueの間、取得する |
| range(int, int) |<u>第２引数の数値-1まで</u>のIntStreamを生成。整数（Int、Long）だけ！DoubleStreamにはない |
| rangeClosed(int, int) |<u>第２引数の数値を含む</u>IntStreamを生成。整数（Int、Long）だけ！DoubleStreamにはない |
| generate(Supplier) | 無限ストリームの生成。 |
| iterate(T, UnaryOperator)、iterate(T, Predicate, UnaryOperator)  | 無限ストリームの生成。for文みたいに使える |

・特殊化ストリーム間での型変換
| メソッド | メモ |
| --- | --- |
| asLongStream() | IntStream ⇒ LongStreamへの変換。**引数なし** |
| asDoubleStream() | IntStream, LongStream ⇒ DoubleStreamへの変換。**引数なし** |
|  |  |

◆コンパレータ

| メソッド | メモ |
| --- | --- |
| static comparing(Function) | ストリーム要素の中身のどれを基準に並べるか指定できる（基準はComparable実装前提） |
| static comparingInt(ToIntFunction) | ストリーム要素の数値型の中身の基準を指定 |
| comp1.thenComparing(comp2) | comp1で並べ替え、次にcomp2で並べ替え |

【map】  
・1→1変換  
・戻り値は値  
・例：map(x -> x * 2)  

【flatMap(Function<T, Stream\<R>>)】  
・1→0以上へ変換  
・戻り値はStream  
・各Streamを連結する  
・例：flatMap(str -> Arrays.stream(str.split(",")))  
　イメージ  
　要素 → Stream生成 → 連結  

【mapMulti(BiConsumer<T, Consumer\<R>>)】  
・1→0以上へ変換  
・**戻り値なし**  
・Consumer.accept()で要素を流す  
・途中でStreamを生成しないのでflatMapより効率が良い場合がある  
・例：  
```
mapMulti((x,c)->{
    c.accept(...);
})
```
　イメージ
　要素 → accept(); accept(); …  

### 終端操作　リダクション（集約）

■Stream<T>インタフェース
| メソッド | メモ |
| --- | --- |
| Optional max(Comparator) | getで最大要素を取得 |
| long count() | 全ストリームで戻り値long |
| reduce |  |
| collect(Collector), <br>collect(Supplier, BiConsumer, BiConsumer) | 可変リダクション |

■特殊化ストリーム
| メソッド | メモ |
| --- | --- |
| OptionalInt IntStream.min() | getAsIntで最小値取得 |
| OptionalDouble IntStream.average() | 全特殊化ストリームで戻り値OptionalDouble。getAsDoubleで平均値取得 |
| int IntStream.sum() | 戻り値の型は特殊化ストリームの基本データ型（DoubleStreamだったらdouble） |
|  |  |
（min()やaverage()がOptional系なのは要素が空の可能性があるから）

■Collectorsクラスのファクトリメソッド

collect(Collector)の中に書くメソッド。staticで、戻り値はCollector型。

| メソッド | メモ |
| --- | --- |
| toList(), toSet(), toCollection(Supplier) | 変更可能なコレクションを返す |
| toMap(Function, Function), toMap(Function, Function, BinaryOperator), toMap(Function, Function, BinaryOperator, Supplier) | マップにして返す。第３引数はキー重複時の操作。第４引数は具体的なマップオブジェクトを指定できる |
| groupingBy | Map<key, List<T>>で返すか、Map<key, D>もある(Dはcountなど集計) |
| pratitionBy | yesかnoで２分割する。Map<Boolean, List<T>>か、Map<Boolean, D>もある(Dはcountなど集計) |

また、Collectorインタフェースの実装クラスを作ってcollectに入れることで独自のリダクションも可能（ただしあまり使われない？）。  
以下メソッドのオーバーライド必須。

| メソッド名（戻り値型） | 役割 |
|---|---|
| `Supplier<A> supplier()` | 結果を格納するための空のコンテナ（入れ物）を生成する |
| `BiConsumer<A, T> accumulator()` | Streamの要素を1つずつコンテナに追加する処理を定義する |
| `BinaryOperator<A> combiner()` | 並列Streamで分割された複数のコンテナの結果を結合する処理を定義する |
| `Function<A, R> finisher()` | 最終的な結果型へ変換する処理を定義する |
| `Set<Collector.Characteristics> characteristics()` | Collectorの特性（並列処理可能か、finisher不要かなど）を返す |
|  |  |

## ４章　モジュールシステム

exports ：外部に公開する「パッケージ名」を指定。
requires：依存する「モジュール名」を指定。

`--module-path （または-p　ただしjdepsコマンドでは短縮不可）`  
`--module （または-m）`  

（↓javaコマンドと一緒に使用。javacとは使えない）  
`--list-modules`  
`--describe-module （または-d）`  
`--show-module-resolution`  

■コンパイル  

複数のモジュールを**個別に**コンパイルするときは、どれもrequires(依存)してないモジュールからコンパイルする。じゃないと、module-info.javaコンパイル時に、requiresに指定したモジュールが見つけられずにコンパイルエラーになる。  
依存するモジュールの場所はモジュールパス`--module-path`を指定する。

一括コンパイルすればOK  
`javac -d out --module-source-path src --module modx,mody,client`  
⇒src直下に各モジュールフォルダがある前提で読み込まれる  
依存しているモジュールは、-mに指定しなくても自動的にコンパイル対象になる  

■実行  

`java --module-path out\client --module client/app.Main`  
⇒classファイル指定で実行する方法。モジュールパスはmodule-infoファイルがある場所を指定。

`java --module-path mlib --module client/app.Main`  
⇒jarファイル指定で実行する方法。モジュールパスはjarファイルがある場所を指定。

■モジュールグラフ  
「どのモジュールが、どのモジュールに依存しているか」を図にしたもの  
例：`moduleA → moduleB → moduleC`  

■jdepsコマンド  
クラスファイルの依存関係を出力するコマンド。  
`jdeps [オプション] --module-path mlib mlib\client.jar`みたいに使う。  
<u>jdepsコマンドでは--module-pathを短縮できない！！</u>
| オプション | メモ |
| --- | --- |
| オプションなし | パッケージレベルでの依存関係を出力 |
| -summary(または-s) | 依存関係を簡略表示 |
| -verbose, -verbose:class, -verbose:package | すべてのクラスレベルの依存関係。verbose(冗長)で覚える |
| -dotoutput (フォルダ名) | 指定したパスのjarの分析結果を(フォルダ名)に、各jarのdotファイルとsummary.dotファイルとして作成する。-summaryオプション付きだとsummary.dotのみ作成。 |
|  |  |

■モジュールの種類  

無名モジュール(クラスパス読み込み)と名前付きモジュールでパッケージが重複したときは**名前付きモジュールが優先**され、クラスパス上のものは無視される。  
名前付きモジュールと自動モジュールでパッケージが重複した場合は**コンパイルエラー**になる。  

無名モジュールからすべてのモジュール参照化だけど、名前付きモジュールからは参照できない（名前がないから明示的にrequiresできない）。自動モジュールからは参照できる。

自動モジュールや無名モジュールから名前付きモジュールを参照するときは、コンパイル時や実行時に`--add-modules`をつける！

非公開にしたパッケージを一時的に公開するには、コンパイル時に `--add-exports 対象モジュール/公開パッケージ=利用モジュール名` をつける！

- 名前付きモジュール  
　・module-info.class<u>あり</u>  
　・モジュールパス--module-pathから読み込み  
　・classファイルでもjarファイルでもOK
- 無名モジュール  
　・module-info.class<u>なし</u>  
　・クラスパス-cpから読み込み  
　・classファイルでもjarファイルでもOK  
　・すべてのパッケージをexport  
- 自動モジュール  
　・module-info.class<u>なし</u>  
　・モジュールパス--module-pathから読み込み  
　・jarファイルじゃなきゃだめ  
　・すべてのパッケージをexport  
　・jarファイル名(例「foo-bar-1.0.1.jar」→「foo.bar」)か、MANIFEST.MFのAutoMatic-Module-Name属性の指定値で名前が自動でつけられる

■モジュール移行の方式  
ボトムアップが安全で確実だが、末端がメンテナンスで使えないときとかはトップダウンを選択する。
- ボトムアップ移行  
1番利用されている側から名前付きモジュールへ移行。使われてるパッケージをjdepsとかで調べて、それらをexportsしていき、それ以外は非公開。
- トップダウン移行
利用している側から名前付きモジュールへ移行。そのとき利用されている側が無名モジュールだと参照できなくなってしまうので、<u>一旦モジュールパス読み取りにして自動モジュールにしておく。</u>

■カスタムランタイムイメージ  
必要な機能だけを詰め込んだ軽量なJava実行環境を自分で作れる仕組み。  
`jlink [オプション] --module-path モジュールパス --add-modules モジュール名 --output 出力先`  
例：`jlink --module-path mlib --add-modules client --output image\clientapp`みたいに使う。そしたらimage\clientappにbinフォルダができて、その中のjava.exeでjavaコマンド使えば、必要最小限の構成でモジュール実行できる。

| オプション | メモ |
| --- | --- |
| --launcher コマンド名=モジュール名/クラス名 | 独自の起動コマンドを作成 |
| --compress=0か1か2 | 圧縮レベル指定 |
|  |  |

### java.util.ServiceLoader
サービス（インターフェース）と、その実装（プロバイダ）を分離して利用する仕組み。  

提供側のjarのMETA-INF/services/(使用インターフェースAPI名)のファイル内に、提供する実装クラス名を書けば、利用者側からServiceLoader\<S>で読み込める。  
利用者側：  
`ServiceLoader<Greeting> loader = ServiceLoader.load(Greeting.class);`  
`for(Greeting obj : loader) { obj.hello(); }`  
みたいに使う。ServiceLoaderはIterableを実装しているので拡張for文が使えて、提供される実装クラスをすべて取り出して使用できる。

module-info.javaがあれば  
提供側：`provides (インターフェース名) with (実装クラス), (実装クラス), …`  
利用側：`requiers (利用モジュール名) uses (インターフェース名)`  
で使える。

## ５章　並列処理

### Threadクラスの主なメソッド

| メソッド | メモ |
| --- | --- |
| Thread.State getState() |  |
| void interrupt() | 指定スレッドに割り込む |
| void join() | 指定スレッドの終了待機する　**InterruptedException例外処理必須** |
| void join(long millis) | 指定スレッドの終了待機する　**InterruptedException例外処理必須** |
| void sleep(long millis) | 中断する　**InterruptedException例外処理必須** |
| static void yield() | 他のスレッドに譲る |
| | |

### 排他制御
'[修飾子] synchronized 戻り値の型 メソッド名() {}'  
'synchronized (ロックを提供するオブジェクト) {}'  
'[修飾子] static synchronized 戻り値の型 メソッド名() {}'  
'synchronized (StaticSample.class) {}'  （staticメソッド内）

'ThreadLocalRandom.current().nextLong(5)'  
現在のスレッドで乱数を生成するクラス。  
current()でThreadLocalRandomのオブジェクト生成、0以上5未満の long 型の乱数を返す。

・アトミック変数  
AtomicBoolean、AtomicInteger、AtomicLong、AtomicReference(参照型)  
メソッドは名前である程度判断できそう。  
boolean compareAndSet(int expectedValue, int newValue)  
　⇒これだけ覚える。値がexpectedValueと等しければnewValueを設定してtrueを返す。    
increment または decrement  
　⇒1ずつ増減するので引数なし  
add  
　⇒引数で渡した数値を足す  
getAndIncrement  
　⇒1足して**増加前**を取得。順番で判断  

### 同期制御

Objectクラスのメソッド↓  
| メソッド | メモ |
| --- | --- |
| final void notify()| 待機中の１スレッドに解除通知 |
| final void notifyAll()| 待機中の全スレッドに解除通知 |
| final void wait() | 通知来るまで待機  **InterruptedException例外処理必須** |
| final void wait(long millis) | 通知来るまでか指定時間待機 **InterruptedException例外処理必須** |
| | |

これらはsynchronizedメソッドorブロック内（オブジェクトロック所有中）に呼び出す。  
所有していないときに使用すると**IllegalMonitorStateException**になる。

### 並行処理コレクション

名前から特徴を把握可能。  
「Blocking～」はスレッドのブロックによりスレッドセーフ実現（上限付き）  
「Concurrent～」はブロックせず同時実行を安全に実現  
「CopyOnWrite～」は書き込み時にコピーを作成  

### Executorフレームワーク

・Executorインタフェース  
実行は'void execute(Runnable command)'を呼ぶ。

・ExecutorServiceインタフェース  

| メソッド | メモ |
| --- | --- |
| Future<?> submit(Runnable task) | タスク送信。正常完了でnullをもつFutureを返す |
| <T> Future<T> submit(Callable<T> task) | タスク送信。結果取得用 |
| shutdown() | タスク受付終了、受付済タスク**実行** |
| shutdownNow() | タスク受付終了、受付済タスクの**中止を試みる(interrupt)**←割り込みを検知しない処理whileとかは止まらない |
| boolean awaitTermination(long, TimeUnit) **InterruptedException例外処理必須** | シャットダウン要求後、待機 |

●'Future<?> submit(Runnable task)'と'<T> Future<T> submit(Callable<T> task)'の違い  
Runnable run()は戻り値がなしなのでFutureから取得できる結果もない。何か型はあるけど気にしなくていい、という意味で<?>。  
Callable call()は戻り値ありなので引数の型パラメータ<T>に応じて、戻り値のFuture<T>型パラメータも同じ型になる。

●ファクトリメソッド  
Executors.newSingleThreadExecutor(); シングルスレッドなので、送信されたタスクは順番実行  
Executors.newFixedThreadPool(2); ←スレッドプール数  
Executors.newCachedThreadPool(); ←スレッドプール数指定なし拡張  

・ScheduledExecutorServiceインタフェース 

●ファクトリメソッド  
Executors.newSingleThreadScheduledExecutor(); シングル  
Executors.newScheduledThreadPool(2); ←スレッドプール数  

| メソッド | メモ |
| --- | --- |
| ScheduledFuture<?> schedule(Runnable, long, TimeUnit) | 指定時間遅延タスク実行 |
| <V> ScheduledFuture<V> schedule(Callable<V>, long, TimeUnit) | 指定時間遅延タスク実行。結果取得用 |
| ScheduledFuture<?> scheduleAtFixedRate(Runnable, long initialDelay, long period, TimeUnit) | 指定時間遅延後にタスク実行、**period周期**で繰り返し |
| ScheduledFuture<?> scheduleWithFixedDelay(Runnable, long initialDelay, long delay, TimeUnit) | 指定時間遅延後にタスク実行、**delay間隔空けて**繰り返し |

AtFixedRateは**開始時刻を基準**に次回実行時刻を決める。  
WithFixedDelayは**前回の処理終了時刻を基準**に待機時間を数える。  
結果は受け取れないのでタスクはCallableじゃなくてRunnable。  
周期で「タスクを送信」なのでshutdownすると新規タスクは受け付けなくなるので以降は受け付けなくなって終了する。  
ただしScheduledFutureにgetしたら実行終了を待機するので止まらない。

・Future<T>

未来(Future)に結果が入る箱。esをsubmitした時点で箱を受け取って、get()でタスク完了後のTを受け取る。

| メソッド | メモ |
| --- | --- |
| get() | 実行待機してから取得。**いろいろスローするから例外処理必須** |
|  |  |

### Flow API

Flow.Publisher<T>.subsclibe(Subscriber)  
呼んだら内部でonSubscribe呼ぶ。パブリッシャーがSubscriberにSubscliption設定する。

（参考）　Publisherがsubscribe内のonSubscribe呼び出しで渡すSubscriptionは、Subscriberごとに違うオブジェクト。

onSubscribeにrequest(1)を書かなかったら、submitしてもなにも受け取らないのでonNext実行されない！ただしonErrorは実行される。

request(n)
→ n個受け取る要求

## ６章　ファイルI／O

### ストリームの分類

|分類|抽象クラス|代表クラス|ノード/フィルタ|用途|
|---|---|---|---|---|
|バイト入力|InputStream|FileInputStream|ノード|バイト読込|
|バイト入力|InputStream|BufferedInputStream|フィルタ|高速化|
|バイト入力|InputStream|ObjectInputStream|フィルタ|オブジェクト読込|
|バイト出力|OutputStream|FileOutputStream|ノード|バイト書込|
|バイト出力|OutputStream|BufferedOutputStream|フィルタ|高速化|
|バイト出力|OutputStream|ObjectOutputStream|フィルタ|オブジェクト保存|
|バイト文字変換|Reader|InputStreamReader|フィルタ|バイト→文字変換|
|文字入力|Reader|FileReader|ノード|文字読込|
|文字入力|Reader|BufferedReader|フィルタ|高速化・readLine()|
|文字出力|Writer|FileWriter|ノード|文字書込|
|文字出力|Writer|BufferedWriter|フィルタ|高速化|
|文字出力|Writer|PrintWriter|フィルタ|printf(), print(), println()|

java.ioパッケージの基底（抽象）クラス  
InputStream, OutputStreamなど、Streamがつくとバイトストリーム  
Reader, Writerなどがつくとキャラクタストリーム  

**ノードストリーム**は単体で使うもの、**フィルタストリーム**はノードストリームと組み合わせて使うもの。  
ノードストリームはFileInput(Output)Stream, FileReader(Writer)だけ覚える。他はだいたいフィルタストリーム  
Object～はバイトストリームだけ(InputStream, OutputStream)。

基本的にストリームはリソースのclose()が必要だが、AutoCloseable実装してたらtry-with-resources文の使用可能。Outputはファイルがなければ新規作成するものならOutputのほうから先に書く。  

#### getBytes()

```java
byte[] b = "abcd".getBytes();
```

結果

```text
{97, 98, 99, 100}
```

文字コードに従って byte[] に変換される。

### FileReader と BufferedReader

|クラス|メソッド|戻り値|
|---|---|---|
|FileReader|read()|int|
|BufferedReader|read()|int|
|BufferedReader|readLine()|String|

FileReaderのread()の戻り値はint型。なぜならread()はEOFの場合に-1返すけどcharだと0~65535で-1を返せないから、仕様！なので結果を(char)cのようにキャストする。

```java
int c;

while((c = fr.read()) != -1){
    System.out.print((char)c);
}
```

FileOutputStreamはflushしなくてもwriteした時点で書かれる（オブジェクトにバッファ持たず、その場でOSに書き込み依頼する）。  
バッファを持つストリームではtry-with-resources使った場合、flush()しなくてもclose()されたら書き込まれる。

フィルタストリーム（BufferedReader／BufferedWriterなど）の引数でバイトストリーム⇔キャラクタストリームはできないので、橋渡しのInputStreamReader／OutputStreamWriterクラスを使う。  
・キャラクタ⇒バイト  
`BufferedReader br = new BufferedReader(new InputStreamReader(System.in));`

### Consoleクラス  

|メソッド|戻り値|エコー|
|---|---|---|
|readLine()|String|入力文字を表示|
|readPassword()|char[]|**常に非表示**|

Consoleクラスはコンストラクタprivateなのでnewできない。System.console()を使う。
ConsoleのreadLineはStringを戻すけど、readPasswordはchar[]を戻す（セキュリティ上の理由。Stringはイミュータブルなのでメモリ上に残り続ける可能性があるため）。

`readPassword()` はエコーのON/OFFを切り替えられないので常に入力文字は表示されない！


```
Console console = System.console();
String id = console.readLine("Enter yout ID     :");
char[] pass = console.readPassword("Enter your password     :");
Arrays.fill(pass, ' '); //←使い終わったらメモリ内容を消す
```
### データの書式設定

■ format()/printf() 書式指示子  

| 書式 | 意味 | 例 |
|------|------|----|
| `%s` | 文字列 | `printf("%s", "Java");` → `Java`  |
| `%d` | 整数 | `printf("%d", 123);` → `123`  |
| `%f` | 小数 | `printf("%f", 3.14);` → `3.140000`<br>（デフォルトで小数点以下６桁）  |
| `%.2f` | 小数2桁 | `printf("%.2f", 3.14159);` → `3.14`  |
| `%c` | 文字 | `printf("%c", 'A');` → `A`  |
| `%b` | 真偽値 | `printf("%b", true);` → `true`  |
| `%n` | 改行 | `printf("A%nB");` → `A`<br>`B`  |
| `%%` | `%`を表示 | `printf("100%%");` → `100%`  |

幅指定

| 書式 | 意味 | 例 |
|------|------|----|
| `%5d` | 右寄せ（幅5） | `printf("%5d", 12);` → `"   12"`  |
| `%-5d` | 左寄せ（幅5） | `printf("%-5d!", 12);` → `"12   !"`  |
| `%05d` | 0埋め | `printf("%05d", 12);` → `00012`  |

### シリアライズ

オブジェクトのシリアライズは、  
・インスタンス変数が対象。static変数は対象外  
・`transient`を指定するとシリアライズ対象外  
・メソッドはデータじゃないので対象外  
・<u>変数が参照型の場合は、そのクラスもSeriarizableインタフェース実装必要</u>

レコードクラスのコンポーネントはfinalなので、シリアライズ・デシリアライズ時のreadObject, whiteObjectは定義しても使われない。  
・シリアライズ時はそのときの状態（コンポーネント）で書き込まれる  
・デシリアライズ時は標準コンストラクタが呼ばれる

<u>ObjectInputStreamは、書き込まれた順にreadObject()する必要あり！</u>  
あとClassNotFoundException投げるので例外処理が必要。

### Pathインタフェース、Pathsクラス

#### 作成

```java
Path p = Paths.get("sample1.txt");
Path p = Path.of("sample2.txt");
```

#### 主なメソッド

|メソッド|説明|
|---|---|
|getFileName()|最後の名前要素（ファイルかディレクトリ）|
|getParent()|親パス。親を持たないパス(ルートか相対パス)なら null を返す！|
|getRoot()|ルート要素を返す（例：`C:\`、`/`）<br>ルート要素を持たない相対パスなら null を返す！|
|getName(int)|<u>ルートを除いた</u>指定インデックスの要素|
|resolve(Path), resolve(String)|単純にパスを結合。引数が絶対パスなら引数のPathを返す|
|resolveSibling(Path), resolveSibling(String)|<u>親要素に</u>引数のパスを結合。引数が絶対パスなら引数のPathを返す|
|relativize()|相対パス生成|
|normalize()|冗長化を排除　`.` `..` を整理する！|
|toAbsolutePath()|絶対パス|

### FileSystem

|メソッド|static|
|---|---|
|getPath()|×|

取得例

```java
FileSystem fs = FileSystems.getDefault();
Path p = fs.getPath("sample.txt");
```

### Filesクラス

Filesクラスはstaticメソッドのみ！！

#### 主なメソッド

|メソッド|メモ|
|---|---|
|Path createDirectories(Path)|ディレクトリを1つ作成（親ディレクトリは自動）|
|Path createDirectory(Path)|ディレクトリを1つ作成<br>（親ディレクトリは自動作成しない。ないとNoSuchFileException）|
|Path createFile(Path)|空ファイルを作成|
|Path copy(Path, Path, オプション)|ファイル・ディレクトリをコピー|
|Path move(Path, Path, オプション)|ファイル・ディレクトリを移動・リネーム|
|void delete(Path)|ファイル・空ディレクトリを削除。削除できなければ例外|
|boolean deleteIfExists(Path)|存在すれば削除し、削除したらtrue|
|List<String> readAllLines(Path)|ファイルの全行を読み込む|
|Path write(Path, byte[])|バイト配列を書き込む|
|Stream<Path> walk(Path)|ディレクトリツリーを深さ優先で走査|
|boolean exists(Path)|ファイル・ディレクトリが存在するかを確認|
|boolean notExists(Path)|ファイル・ディレクトリが存在しないかを確認|

### Filesクラスの主な例外

|メソッド|主な例外|
|---|---|
|createFile()|FileAlreadyExistsException|
|createDirectory()|FileAlreadyExistsException|
|copy()|FileAlreadyExistsException|
|move()|FileAlreadyExistsException|
|delete()|NoSuchFileException、DirectoryNotEmptyException|

#### Filesクラスのファイルツリー探索

|メソッド|メモ|
|---|---|
|Stream<Path> list(Path)|<u>指定フォルダ直下のみ探索</u>|
|Stream<Path> find(Path, int, BiPredicate<Path, BasicFileAttributes>)|条件に一致するファイル・ディレクトリを探索する。walk() + 条件判定のような処理|
|Stream<Path> walk(Path, FileVisitOption...)|指定したパスを起点にファイルツリーを探索する。**深さ優先探索**。ファイル・ディレクトリ両方を対象にする。シンボリックリンクを辿るなどの探索オプションを指定できる|
|Stream<Path> walk(Path, int, FileVisitOption...)|探索の最大深度を指定|
|**Path** walkFileTree(Path, FileVisitor<? super Path>)|再帰的に辿り、みつかったら処理。<br>FileVisitor<T>インタフェースもしくはSimpleFileVisitorクラスを実装して見つかった場合の処理を書く（無名クラスでOK）。|

#### シンボリックリンク

- 別ファイル・別ディレクトリへのリンク
- Windowsのショートカットより実体に近い存在

## 見ておく資料

java クラスOptional<T>
https://docs.oracle.com/javase/jp/11/docs/api/java.base/java/util/Optional.html