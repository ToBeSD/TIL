## 제네릭(Generics)
제네릭은 클래스나 메서드에서 사용할 데이터 타입을 컴파일 시점에 지정할 수 있도록 하는 기능이다. 이를 통해 타입 안정성을 보장하고, 형 변환을 최소화할 수 있다.

### 제네릭을 사용하는 이유
1. 타입 안정성 보장
컴파일 시점에서 타입을 체크하여 런타임 타입 오류를 방지할 수 있다.
2. 형 변환 제거
기존에는 Object 타입을 사용하여 다운캐스팅해야 했지만, 제네릭을 사용하면 불필요한 형 변환을 줄일 수 있다.
3. 재사용성 증가
다양한 타입에 대해 동작하는 범용적인 코드 작성이 가능하다.

## 제네릭의 사용법

### 1. 제네릭 클래스

```
class Box<T> { // <T> : 타입 파라미터
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
        Box<String> stringBox = new Box<>(); // T -> String으로 지정
        stringBox.set("Hello");
        System.out.println(stringBox.get()); // Hello

        Box<Integer> intBox = new Box<>(); // T -> Integer로 지정
        intBox.set(100);
        System.out.println(intBox.get()); // 100
    }
}

```

### 2. 제네릭 메서드

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

### 3. 제네릭 타입 제한 (`<T extends Number>`)

제네릭 클래스나 메서드를 정의할 때 타입 매개변수(T)의 범위를 제한하는 방식이다.

```
class NumberBox<T extends Number> { // Number 또는 그 하위 클래스만 허용
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
        // NumberBox<String> strBox = new NumberBox<>(); // 컴파일 오류!
    }
}
```

##### 📌 특징
- T는 Number 또는 그 하위 타입으로 고정됨.
- "클래스를 설계할 때" 제한하는 방식.
- T는 고정된 단일 타입이므로, setValue()에서 T 타입 추가 가능.

### 4. 와일드카드(?)

#### 상한 경계 와일드카드 : `<? extends T>` T 또는 T의 하위 타입 허용 (읽기 전용)

컬렉션을 사용할 때, 읽기 전용(Producer)으로 제한하고 싶을 때 사용된다.

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

##### 📌 특징
- `? extends Number`는 Number의 하위 타입이 들어올 수 있음을 의미.
- `List<? extends Number>`는 `List<Integer>`, `List<Double>` 등을 받을 수 있음.
- 정확한 타입이 불확실하기 때문에 값을 추가할 수 없음.

#### 값 추가(쓰기)가 불가능한 이유

컴파일러 입장에서 리스트의 실제 타입이 T의 하위 타입 중 무엇인지 알 수 없기 때문이다.

```
List<? extends Number> list = new ArrayList<Integer>();

list.add(10);  // ❌ 컴파일 에러!
```

`list`는`< ? extends Number>`이므로 `Number`의 어떤 하위 타입이 올지 알 수 없다.
현재는 `list = new ArrayList<Integer>();`라고 명확하지만,
다른 곳에서 `list = new ArrayList<Double>();`로 변경될 수도 있다.
만약 `list.add(10);`을 허용하면, `list`가`ArrayList<Double>`일 때 **잘못된 타입(Integer)** 이 들어가는 문제가 발생할 수 있다.
Java는 이를 방지하기 위해 애초에 값 추가를 막아버린다.

그래도 null은 추가할 수 있다.
```
List<? extends Number> list = new ArrayList<Integer>();
list.add(null);  // ✅ 가능
```
null은 어떤 타입이든 될 수 있기 때문에 예외적으로 허용된다.


#### 하한 경계 와일드카드 : `<? super T>` T 또는 T의 상위 타입 허용 (쓰기 가능)

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

리스트에 값을 추가할 수 있지만, 값을 읽을 때는 Object 타입으로 읽어야 한다.

#### 값 읽을 때 Object로 나오는 이유
`numList.get(0);`의 반환 타입은 `Object`이다.
이유는 `<? super Integer>`가 `Integer`보다 더 상위 타입이 될 수도 있기 때문
(예: `List<Number>`라면 `Number` 타입이 들어 있을 수도 있음)
Java는 타입 안정성을 보장해야 하므로, 가장 확실한 타입인 `Object`로 리턴.

## 제네릭 타입 제한 VS 상한 경계 와일드카드

| 비교 항목  | **제네릭 타입 제한** (`<T extends Number>`) | **상한 경계 와일드카드** (`? extends Number`) |
|-----------|--------------------------------|------------------------------|
| **적용 대상** | 제네릭 클래스 / 메서드 정의 | 컬렉션 (리스트, 맵 등) |
| **사용 시점** | 클래스를 정의할 때 | 메서드 파라미터에서 사용 |
| **값 추가 가능 여부** | ✅ `T` 타입 추가 가능 | ❌ 추가 불가능 |
| **값 읽기 가능 여부** | ✅ `T` 타입으로 읽기 가능 | ✅ `Number` 타입으로 읽기 가능 |
| **사용 목적** | 타입을 확정적으로 제한 | 특정 타입을 읽기 전용으로 사용 |

## 제네릭의 한계

### 기본 타입(Primitive Type) 사용 불가

Box<int>처럼 기본 타입은 사용할 수 없음. 대신 Integer, Double 같은 Wrapper 클래스를 사용해야 한다.

### 정적(static) 변수에 사용 불가

제네릭은 객체 생성 시점에 타입이 결정되므로, 클래스 수준의 static 변수에는 사용할 수 없음.

### 런타임 타입 정보 부족 (Type Erasure)

제네릭은 컴파일 시 타입 정보를 제거하기 때문에 런타임에는 List<String>과 List<Integer>가 동일한 List로 인식됨.
즉, instanceof로 List<String>인지 확인할 수 없음.
