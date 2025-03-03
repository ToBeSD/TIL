## ジェネリックス(Generics)
ジェネリックスは、クラスやメソッドで使用するデータ型をコンパイル時に指定できる機能です。これにより、型の安全性を保証し、型変換を最小限に抑えることができます。

### ジェネリックスを使用する理由
1. **型の安全性の保証**  
   コンパイル時に型をチェックすることで、ランタイム時の型エラーを防ぐことができます。  
2. **型変換の不要**  
   以前は`Object`型を使用してダウンキャストする必要がありましたが、ジェネリックスを使用すると不要になります。  
3. **再利用性の向上**  
   様々な型に対応できる汎用的なコードを記述できます。  

## ジェネリックスの使い方

### 1. ジェネリックスを使ったクラス

```
class Box<T> { // <T> : 型パラメータ
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}

public class Main {
    public static void main(String[] args) {
        Box<String> stringBox = new Box<>(); // T → String に指定
        stringBox.set("Hello");
        System.out.println(stringBox.get()); // Hello

        Box<Integer> intBox = new Box<>(); // T → Integer に指定
        intBox.set(100);
        System.out.println(intBox.get()); // 100
    }
}
```

### 2. ジェネリックスを使ったメソッド

```
class Util {
    public static <T> void print(T value) {
        System.out.println(value);
    }
}

public class Main {
    public static void main(String[] args) {
        Util.print("Hello");  // String
        Util.print(123);      // Integer
        Util.print(3.14);     // Double
    }
}
```

### 3. 境界型パラメータ(`<T extends Number>`)

ジェネリックスの型パラメータ`T`の範囲を制限することができます。

```
class NumberBox<T extends Number> { // Number またはそのサブクラスのみ許可
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}

public class Main {
    public static void main(String[] args) {
        NumberBox<Integer> intBox = new NumberBox<>();  // OK
        NumberBox<Double> doubleBox = new NumberBox<>(); // OK
        // NumberBox<String> strBox = new NumberBox<>(); // コンパイルエラー！
    }
}
```

#### 📌 特徴
- `T`は`Number`またはそのサブタイプに限定されます。
- `T`は固定された単一の型であるため、`setValue()`で`T`型の値を追加可能です。

## 4. ワイルドカード(?)

### 上限境界ワイルドカード(<`? extends T`>)
`T`または`T`のサブタイプを許可（読み取り専用）

コレクションを使用する際に、読み取り専用として制限する場合に使用します。

```
class Printer {
    public static void printList(List<? extends Number> list) {
        for (Number n : list) {
            System.out.println(n);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        List<Integer> intList = List.of(1, 2, 3);
        List<Double> doubleList = List.of(1.1, 2.2, 3.3);
        
        Printer.printList(intList);   // OK
        Printer.printList(doubleList); // OK
    }
}
```

##### 📌 特徴
- `<? extends Number>`は`Number`のサブタイプを受け入れます。
- `List<? extends Number>`は`List<Integer>`や`List<Double>`などを受け取れます。
- 正確な型が不明なため、値の追加は不可です。

#### 値の追加（書き込み）が不可能な詳しい理由
コンパイラはリストの具体的な型が`T`のサブタイプのどれなのかを特定できません。

```
List<? extends Number> list = new ArrayList<Integer>();

list.add(10);  // ❌ コンパイルエラー！
```

`list`は`<? extends Number>`のため、どのサブタイプが来るか分かりません。
例えば`list = new ArrayList<Double>();`となる場合もあります。
もし`list.add(10);`を許可すると、`list`が`ArrayList<Double>`の時に異なる型（Integer）を追加することになります。
この問題を防ぐためにJavaは値の追加を禁止しています。

#### ただし`null`の追加は可能です。

```
List<? extends Number> list = new ArrayList<Integer>();
list.add(null);  // ✅ OK
```
`null`はどの型にもなれるため、例外的に許可されます。


### 下限境界ワイルドカード(`<? super T>`)
`T`または`T`の親クラスを許可（書き込み可能）

```
class Adder {
    public static void addNumber(List<? super Integer> list) {
        list.add(10); // OK
    }
}

public class Main {
    public static void main(String[] args) {
        List<Number> numList = new ArrayList<>();
        Adder.addNumber(numList); // OK

        List<Object> objList = new ArrayList<>();
        Adder.addNumber(objList); // OK
    }
}
```

リストに値を追加できますが、値を読み取る際には`Object`型として取得する必要があります。

#### 値を`Object`として読み取る理由

```
Object obj = numList.get(0);
```

`numList.get(0);`の戻り値の型は`Object`です。  
理由は、`<? super Integer>`が`Integer`よりも上位の型になる可能性があるためです。  
例えば、`List<Number>`の場合、`Number`型の値が含まれている可能性があリます。
Javaは型の安全性を保証する必要があるため、最も確実な型である`Object`を返すようになっています。

## ジェネリックス型制限 VS 上限境界ワイルドカード

形が似たような感じで、何が違うのかわかりやすく表で整理しました。

| 比較項目  | **ジェネリックス型制限** (`<T extends Number>`) | **上限境界ワイルドカード** (`? extends Number`) |
|-----------|--------------------------------|------------------------------|
| **値の追加可否** | ✅ `T` 型の追加可能 | ❌ 追加不可 |
| **値の読み取り可否** | ✅ `T` 型として読み取り可能 | ✅ `Number` 型として読み取り可能 |
| **使用目的** | 型を確定的に制限 | 特定の型を読み取り専用で使用 |

## ジェネリックスの限界

### プリミティブ型の使用不可

`Box<int>`のようにプリミティブ型は使用できない。その代わりに`Integer`や`Double`のようなラッパークラスを使用する必要があります。

### `static`変数での使用不可

ジェネリックスはオブジェクトの生成時に型が決定されるため、クラスレベルの`static`変数には使用できません。

### 実行時の型情報不足（型消去）

ジェネリックスはコンパイル時に型情報が削除されるため、実行時には `List<String>` と `List<Integer>` は同じ `List` として認識される。
つまり、`instanceof` を使って `List<String>` であるかを確認することはできない。
