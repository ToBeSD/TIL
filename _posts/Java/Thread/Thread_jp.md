## マルチスレッド

### マルチスレッド（Multi Thread）とは？
スレッドはプロセス内で実行される最小単位の実行フローです。  
1つのプロセスは複数のスレッドを持つことができ、これを **マルチスレッド** と呼びます。  
マルチスレッドを使用すると、CPUの利用率を向上させ、応答性を改善し、大規模な並列処理を効率的に処理できます。

## マルチスレッドの使用方法

### 1. 直接スレッドを生成する（Thread クラスを継承または Runnable を実装）

```
class MyThread extends Thread {
    public void run() {
        System.out.println(Thread.currentThread().getName() + " 実行中");
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();

        t1.start();
        t2.start();
    }
}
```

#### 特徴

- スレッドを直接生成すると、必要なたびに新しいスレッドオブジェクトが作成されるため、リソースが非効率的に使用される可能性がある  
- 多くのリクエストが同時に発生すると、システムが過負荷になる可能性がある  

### Executorsを使用したスレッドプール方式

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(2);

        for (int i = 0; i < 5; i++) {
            executor.execute(() -> System.out.println(Thread.currentThread().getName() + " 실행 중"));
        }

        executor.shutdown();
    }
}
```

#### 特徴
- Executorsは内部的にスレッドプールを管理し、不必要なスレッドの生成を防ぐ  
- 一定数のスレッドを再利用するため、パフォーマンスが最適化される  
- しかし、execute() を呼び出しても実行されるまで待機する可能性がある（スレッド数に制限があるため）  

| 区分         | 直接スレッド生成        | Executors使用         |
|------------|---------------------|---------------------|
| **スレッド数** | 必要なときに新しく生成 | 一定数のスレッドを再利用 |
| **パフォーマンス** | スレッドが増えると非効率的 | リソース管理が最適化 |
| **制御**     | `start()`、`join()` を直接呼び出す | `execute()`、`shutdown()` を活用 |
| **使用例**   | シンプルなマルチスレッド実行 | 大量の並列処理 |

## マルチスレッドの同期問題  

マルチスレッド環境では、複数のスレッドが同時に共有リソースにアクセスするため、  
⚠ **データ競合（Data Race）** や **整合性の問題** が発生する可能性がある。  
このような問題を **同期問題** と呼ぶ。  

### 同期問題の原因  
**共有リソース（Shared Resource）**  

共有リソースとは、複数のスレッドが同時にアクセスできる変数やオブジェクトを指す。  
例えば、銀行口座から出金を処理するコードを見てみよう。  


```
class BankAccount {
    private int balance = 100;

    public synchronized void withdraw(int amount) {
        if (balance >= amount) {
            System.out.println(Thread.currentThread().getName() + " 출금: " + amount);
            balance -= amount;
            System.out.println(Thread.currentThread().getName() + " 잔액: " + balance);
        } else {
            System.out.println(Thread.currentThread().getName() + " 출금 실패, 잔액 부족");
        }
    }
}

public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();
        
        Runnable task = () -> {
            account.withdraw(50);
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);

        t1.start();
        t2.start();
    }
}
```

#### 問題発生の可能性  
残高が100円のとき、t1とt2が同時に `withdraw(50)` を呼び出した場合  
t1が `if` 文を実行する時点では `balance == 100`  
t2も `if` 文を実行する時点では `balance == 100`  
両方のスレッドが同時に `balance -= 50` を実行すると？  
残高が0円ではなく50円残るという異常な状態が発生！  

### 解決方法  

#### 1. synchronized キーワードの使用  
`synchronized` を使用すると、特定のメソッドやブロックを一度に1つのスレッドのみが実行できる。  

```
class BankAccount {
    private int balance = 100;

    public synchronized void withdraw(int amount) {
        if (balance >= amount) {
            System.out.println(Thread.currentThread().getName() + " 출금: " + amount);
            balance -= amount;
            System.out.println(Thread.currentThread().getName() + " 잔액: " + balance);
        } else {
            System.out.println(Thread.currentThread().getName() + " 출금 실패, 잔액 부족");
        }
    }
}
```

✅ synchronized が付いたメソッドは、一度に一つのスレッドしかアクセスできない  

#### 2. synchronized ブロックの使用  
メソッド全体を同期するとパフォーマンスの低下が発生する可能性がある。  
そのため、共有リソースにアクセスする部分のみを同期するのが望ましい。  


```
class BankAccount {
    private int balance = 100;

    public void withdraw(int amount) {
        synchronized (this) {
            if (balance >= amount) {
                System.out.println(Thread.currentThread().getName() + " 출금: " + amount);
                balance -= amount;
                System.out.println(Thread.currentThread().getName() + " 잔액: " + balance);
            } else {
                System.out.println(Thread.currentThread().getName() + " 출금 실패, 잔액 부족");
            }
        }
    }
}

```

✅ `synchronized (this)` ブロックを使用して必要な部分のみを同期 → パフォーマンス最適化  

#### 3. ReentrantLock の使用  
`ReentrantLock` を使用すると、`synchronized` よりも細かい制御が可能になる。  

```
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class BankAccount {
    private int balance = 100;
    private final Lock lock = new ReentrantLock();

    public void withdraw(int amount) {
        lock.lock();
        try {
            if (balance >= amount) {
                System.out.println(Thread.currentThread().getName() + " 출금: " + amount);
                balance -= amount;
                System.out.println(Thread.currentThread().getName() + " 잔액: " + balance);
            } else {
                System.out.println(Thread.currentThread().getName() + " 출금 실패, 잔액 부족");
            }
        } finally {
            lock.unlock();
        }
    }
}
```

✅ `lock.lock()` と `lock.unlock()` を明示的に呼び出すことで、正確な同期が可能  
✅ `try-finally` を使用して、例外が発生しても必ずロックを解除  
