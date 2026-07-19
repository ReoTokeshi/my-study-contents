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
- 

・ワイルドカードについて  

変数宣言時の型にワイルドカード使用可能。ただし  
インスタンス生成側は具体的な型を指定する必要あるので使用不可！  
'Queue<? super Number> queue = new ArrayDeque<? super Number>();' ⇒NG
'Queue<? super Number> queue = new ArrayDeque<>();' ⇒これはOK  

**PECS (Producer Extends, Consumer Super)** の考え方 

|宣言|コレクションに追加できるもの|取り出せる型|
|--|--|--|
|? extends T|基本的に追加できない（null以外）|T|
|? super T|Tとそのサブクラス|Object|
||||

例： 
・'<? extends Number>'で宣言した場合  
⇒実際の中身がIntegerなのかDoubleなのか判断できないのでコンパイル時点で追加を許可しない。  
⇒実際の中身がNumber以下であることは確定しているため、取り出しはNumberで受けられる。  
・'<? super Number>'で宣言した場合  
⇒実際の中身がNumber以上であることが確定しているため、Number以下の追加は可能。  
⇒実際の中身がNumberなのかObjectなのか判断できないので、取り出しはObjectでのみ受けられる。  

◆ジェネリックメソッド  
・`public static <E> void bar() { }`の順番（戻り値の前）。  
・呼び出し側での型指定方法は、  
`obj.<Integer>foo();`や`List<Integer> list1 = obj.foo();`（obj.<>foo()はダメ）でOK。

◆コレクション

| クラス・インターフェース | メソッド | メモ |
| --- | --- | --- |
| Arrays | List\<E> asList(…) | (Collectionsクラスには無い！)。戻り値の型はArrayListではなくList！！ |
| List\<E> | List\<E> subList(int, int) | 切り出してリスト作成。元のリストが変更不可(List.of生成とか)だと切り出したリストも変更不可。subSetやsubMapもあるぽい |
|  |  |  |
|  |  |  |

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

Setは重複NG。nullはHashSet、LinkedHashSetでは1個だけ入る。TreeSetはだめ（例外）  

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

## ３章　JavaストリームAPI

| メソッド | メモ |
| --- | --- |
| dropWhile(Predicate) | trueの間、スキップして捨てる |
| takeWhile(Predicate) | trueの間、取得する |
| range(int, int) |<u>第２引数の数値-1まで</u>のIntStreamを生成 |
| rangeClosed(int, int) |<u>第２引数の数値を含む</u>IntStreamを生成 |
|  |  |

### 終端操作　リダクション（集約）


■Stream<T>インタフェース
| メソッド | メモ |
| --- | --- |
| Optional max(Comparator) | getで最大要素を取得 |
| long count() | 全ストリームで戻り値long |

■特殊化ストリーム
| メソッド | メモ |
| --- | --- |
| OptionalInt IntStream.min() | getAsIntで最小値取得 |
| OptionalDouble IntStream.average() | 全特殊化ストリームで戻り値OptionalDouble。getAsDoubleで平均値取得 |
| int IntStream.sum() | 戻り値の型は特殊化ストリームの基本データ型（DoubleStreamだったらdouble） |
|  |  |
（min()やaverage()がOptional系なのは要素が空の可能性があるから）

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

mapMulti(BiConsumer)  
1要素→0～複数要素へ変換  
flatMapより中間Listを作らない

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

無名モジュール(クラスパス読み込み)と名前付きモジュールでパッケージが重複したときは**名前付きモジュールが優先**され、クラスパス上のものは無視される。  
名前付きモジュールと自動モジュールでパッケージが重複した場合は**コンパイルエラー**になる。  

■実行  

`java --module-path out\client --module client/app.Main`  
⇒classファイル指定で実行する方法。モジュールパスはmodule-infoファイルがある場所を指定。

`java --module-path mlib --module client/app.Main`  
⇒jarファイル指定で実行する方法。モジュールパスはjarファイルがある場所を指定。

■モジュールグラフ  
「どのモジュールが、どのモジュールに依存しているか」を図にしたもの  
例：`moduleA → moduleB → moduleC`  

■カスタムランタイムイメージ  
必要な機能だけを詰め込んだ軽量なJava実行環境を自分で作れる仕組み。  
jlink コマンドで作成。

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

onSubscribeにrequest(1)を書かなかったら、submitしてもなにも受け取らないのでonNext実行されない！ただしonErrorは実行される。

request(n)
→ n個受け取る要求