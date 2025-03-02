## 멀티 스레드 

### 멀티 스레드(Multi Thread)란?
스레드는 프로세스 내에서 실행되는 가장 작은 단위의 실행 흐름이다. 하나의 프로세스는 여러 개의 스레드를 가질 수 있으며, 이들을 멀티스레드 라고 한다. 멀티스레드를 사용하면 CPU 활용도를 높이고, 응답성을 개선하며, 대규모 병렬 작업을 효율적으로 처리할 수 있다.


## 멀티스레드 사용 방법

### 1. 직접 스레드 생성 (Thread 클래스를 상속하거나 Runnable 구현)

```
class MyThread extends Thread {
    public void run() {
        System.out.println(Thread.currentThread().getName() + " 실행 중");
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

#### 특징

- 스레드를 직접 생성하면 필요할 때마다 새로운 스레드 객체가 만들어지므로 자원이 비효율적으로 사용될 수 있음
- 많은 요청이 동시에 발생하면 시스템이 과부하될 가능성이 있음

### Executors를 사용한 스레드 풀 방식

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

#### 특징
- Executors는 내부적으로 스레드 풀을 관리하여 불필요한 스레드 생성을 방지
- 일정한 개수의 스레드를 재사용하기 때문에 성능이 최적화됨
- 하지만, execute()를 호출해도 실행될 때까지 대기할 수 있음 (스레드 개수 제한이 있기 때문)

| 구분       | 직접 스레드 생성        | Executors 사용         |
|------------|---------------------|---------------------|
| **스레드 개수** | 필요할 때마다 새로 생성 | 일정 개수의 스레드 재사용 |
| **성능**     | 스레드가 많아지면 비효율적 | 자원 관리 최적화 |
| **제어**     | 직접 `start()`, `join()` 호출 | `execute()`, `shutdown()` 활용 |
| **사용 예시** | 단순한 멀티스레드 실행 | 대량의 병렬 작업 처리 |


## 멀티스레드 동기화 문제

멀티스레드 환경에서는 여러 개의 스레드가 동시에 공유 자원에 접근하기 때문에,
⚠ 데이터 충돌(Data Race)과 일관성 문제가 발생할 수 있다.
이러한 문제를 동기화 문제라고 한다.

### 동기화 문제의 원인
**공유 자원 (Shared Resource)**

공유 자원은 여러 스레드가 동시에 접근할 수 있는 변수나 객체를 의미한다.
예를 들어, 은행 계좌에서 출금을 처리하는 코드를 보자.

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

#### 문제 발생 가능성
잔액이 100원일 때, t1, t2가 동시에 withdraw(50)을 호출할 경우
t1이 if 문을 실행할 때 balance == 100
t2도 if 문을 실행할 때 balance == 100
두 스레드가 동시에 balance -= 50을 실행하면?
잔액이 0원이 아니라 50원이 남는 비정상적인 상태 발생!

### 해결방법

#### 1. synchronized 키워드 사용
synchronized를 사용하면 특정 메서드나 블록을 한 번에 한 스레드만 실행할 수 있다.

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
✅ synchronized가 붙은 메서드는 한 번에 한 스레드만 접근 가능


#### 2. synchronized 블록 사용
메서드 전체를 동기화하면 성능 저하가 발생할 수 있다.
따라서 공유 자원에 접근하는 부분만 동기화하는 것이 좋다.

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
✅ synchronized (this) 블록을 사용하여 필요한 부분만 동기화 → 성능 최적화


#### 3. ReentrantLock 사용
ReentrantLock을 사용하면 synchronized보다 더 세밀한 제어가 가능하다

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
✅ lock.lock()과 lock.unlock()을 명시적으로 호출 → 정확한 동기화 가능
✅ try-finally를 사용해 예외가 발생해도 반드시 lock을 해제