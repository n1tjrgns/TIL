**Synchronized 키워드**

- 단순하게는 multi-thread로 동시접근되는것을 막는다.



먼저 Thread는 class의 멤버변수의 자원에 접근할 수 있다.

멤버변수가 Heap 메모리를 사용하기 때문에 가능하다. (이 부분에 대해서는 따로 정리)

Thread가 공유자원에 접근하는 경우 동기화를 해 줘야 할 필요가 있다. (다른 이유들도 많음)



**Synchronized 함수의 사용**

1. synchronized 함수

 

**예제1)**

```java
public class BasicSynchronization {
    private String mMessage;

    public static void main(String[] agrs) {
        BasicSynchronization temp = new BasicSynchronization();
        System.out.println("Test start!");
        new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                temp.callMe("Thread1");
            }
        }).start();
        new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                temp.callMe("Thread2");
            }
        }).start();
        System.out.println("Test end!");
    }

    public synchronized void callMe(String whoCallMe) {
        mMessage = whoCallMe;
        try {
            long sleep = (long) (Math.random() * 100);
            Thread.sleep(sleep);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        if (!mMessage.equals(whoCallMe)) {
            System.out.println(whoCallMe + " | " + mMessage);
        }
    }
}

```

main 함수에서 두개의 thread를 만들어 callMe 함수를 호출한다.

callMe() 함수는 아래와 같다.

1. parameter 값을 멤버변수에 저장
2. 랜덤값 만큼 sleep
3. if 멤버변수와 parameter 값이 같지 않으면 sysout 



> 위 함수에서는 절대 sysout이 출력되지 않는다.

하지만, sychronized 함수를 제거하면 출력된다!



Main 함수를 수정 해보자

```java
public class BasicSynchronization {
    private String mMessage;

    public static void main(String[] agrs) {
        BasicSynchronization temp1 = new BasicSynchronization();
        BasicSynchronization temp2 = new BasicSynchronization();
        System.out.println("Test start!");
        new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                temp1.callMe("Thread1");
            }
        }).start();
        new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                temp2.callMe("Thread2");
            }
        }).start();
        System.out.println("Test end!");
    }

    public void callMe(String whoCallMe) {
        mMessage = whoCallMe;
        try {
            long sleep = (long) (Math.random() * 100);
            Thread.sleep(sleep);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        if (!mMessage.equals(whoCallMe)) {
            System.out.println(whoCallMe + " | " + mMessage);
        }
    }
}

```

temp1, temp2 2개의 객체를 만들어서 각각의 스레드에서 callMe 함수를 호출했다.

또한 callMe 함수 앞의 synchronized 함수를 제거했다.



이 경우에는 스레드가 호출하는 객체 자체가 아예 다르기 때문에 callMe 함수에서 sysout을 출력하지 않는다.



`함수에 synchronized를 걸어주면 그 함수가 포함된 해당 객체(this)에 lock을 거는 것`과 같다.



**예제2)**

```java
public class MyHero {
    private String mHero;

    public static void main(String[] agrs) {
        MyHero tmain = new MyHero();
        System.out.println("Test start!");
        new Thread(() -> {
            for (int i = 0; i < 1000000; i++) {
                tmain.batman();
            }
        }).start();
        new Thread(() -> {
            for (int i = 0; i < 1000000; i++) {
                tmain.superman();
            }
        }).start();
        System.out.println("Test end!");
    }

    public synchronized void batman() {
        mHero = "batman";
        try {
            long sleep = (long) (Math.random() * 100);
            Thread.sleep(sleep);
            if ("batman".equals(mHero) == false) {
                System.out.println("synchronization broken");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public synchronized void superman() {
        mHero = "superman";
        try {
            long sleep = (long) (Math.random() * 100);
            Thread.sleep(sleep);
            if ("superman".equals(mHero) == false) {
                System.out.println("synchronization broken");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

synchronized 함수가 두개인 MyHero 클래스를 생성했다.

두 개의 스레드가 synchronized 함수인 batman()과 superman()을 각각 백만번 호출한다.

batman, superman 함수는 mHero란 멤버변수에 각각 다른 값을 세팅하고, 랜덤하게 sleep한 후에 값이 변했는지 체크한다.

값이 변하면 synchronization broken를 출력한다.



하지만 객체에 lock을 걸었기 때문에 sysout은 출력되지 않는다.



두개의 스레드가 각각의 다른 synchronized함수를 호출하지만 객체에 lock이 걸려있어 동시에 호출할 수 없다.



이 경우에는 synchronized로 인해 객체에 포함된 다른 모든 접근 까지도 lock이 걸리기 때문에 synchronized block이 존재한다.



**Synchronized block**

synchronized 함수의 문제점

1. synchronized 함수가 lock이 걸린다.
2. synchronized 함수를 포함한 객체(this)에 lock이 걸린다.



- 쉽게 말해서 Synchronized를 부분적으로 내가 원하는 부분에만 걸어주고 싶을 때!! 사용하면 된다.



1번 해결 방법

```java
public class SyncBlock1 {
    public ArrayList<Integer> mList = new ArrayList<>();

    public static void main(String[] agrs) throws InterruptedException {
        SyncBlock1 syncblock1 = new SyncBlock1();
        System.out.println("Test start!");
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                syncblock1.add(i);
            }
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                syncblock1.add(i);
            }
        });
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(syncblock1.mList.size());
        System.out.println("Test end!");
    }

    public void add(int val) { 
        /* * Code for synchronization is not needed * */
        synchronized (this) {
            if (mList.contains(val) == false) {
                mList.add(val);
            }
        }
    }
} 

```

syncblock1 이라는 객체를 만든다.

add () 함수를 보면, 내부에 필요한 부분만 synchronized (this) 로 블락 처리를 했다.

두 개의 스레드로 동시에 생성한 add 함수를 호출한다.



sysout의 synblock1.mList.size() 를 보면 10000이 출력된다.



동기화가 필요없는 코드는 주석처리 된 공간에 작성하면 된다.



주석처리 된 부분에 별도의 코드를 작성하지 않는다면 같다고 볼 수 있다.

```java
public synchronized void add() {
//Do something 
} 


public void add() {
	synchronized(this) {
    	//Do something 
   	} 
}
```



Synchronized block에 인자 값은 lock을 걸 대상이다.

따라서 아래처럼 (this)로 표기하면 해당 객체 안에 있는 모든 synchronized block에 lock이 걸린다.



한마디로 여러 쓰레드가 들어와서 각기 다른 synchronized 부분을 호출해도 아래 코드는 this 라는 lock을 사용하기 때문에 기다려야 한다.

```java
public class SyncBlock2 {
    private HashMap<String, String> mMap1 = new HashMap<>();
    private HashMap<String, String> mMap2 = new HashMap<>();

    public static void main(String[] agrs) {
        SyncBlock2 syncblock2 = new SyncBlock2();
        System.out.println("Test start!");
        new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                syncblock2.put1("A", "B");
                syncblock2.get2("C");
            }
        }).start();
        new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                syncblock2.put2("C", "D");
                syncblock2.get1("A");
            }
        }).start();
        System.out.println("Test end!");
    }

    public void put1(String key, String value) {
        synchronized (this) {
            mMap1.put(key, value);
        }
    }

    public String get1(String key) {
        synchronized (this) {
            return mMap1.get(key);
        }
    }

    public void put2(String key, String value) {
        synchronized (this) {
            mMap1.put(key, value);
        }
    }

    public String get2(String key) {
        synchronized (this) {
            return mMap2.get(key);
        }
    }
}

```

put1, get1, put2, get2 를 가지고 있는 SyncBlock2 객체를 생성한다.

두개의 hashmap을 가지고 있다.

네 개의 함수 내부에 동기화가 필요한 부분을 synchronized(this)로 묶는다.

두 개의 쓰레드로 서로 번갈아 가면서 네 개의 함수를 10000번 호출한다.



이 경우에도 네 개의 메소드 안의 synchronized(this) block에 들어가는 순간 자원을 선점하고 lock을 걸어둔다.

따라서 해당 lock이 풀릴때 까지 대기해야 한다.



하지만 hashmap을 동시에 접근하는 경우라고 한다면 아래와 같이 수정하면 훨씬 효율적인 코드가 된다.

```java
public class SyncBlock2 {
    private HashMap<String, String> mMap1 = new HashMap<>();
    private HashMap<String, String> mMap2 = new HashMap<>();
    private final Object object1 = new Object();
    private final Object object2 = new Object();

    public static void main(String[] agrs) {
        SyncBlock2 syncblock2 = new SyncBlock2();
        System.out.println("Test start!");
        new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                syncblock2.put1("A", "B");
                syncblock2.get2("C");
            }
        }).start();
        new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                syncblock2.put2("C", "D");
                syncblock2.get1("A");
            }
        }).start();
        System.out.println("Test end!");
    }

    public void put1(String key, String value) {
        synchronized (object1) {
            mMap1.put(key, value);
        }
    }

    public String get1(String key) {
        synchronized (object1) {
            return mMap1.get(key);
        }
    }

    public void put2(String key, String value) {
        synchronized (object2) {
            mMap2.put(key, value);
        }
    }

    public String get2(String key) {
        synchronized (object2) {
            return mMap2.get(key);
        }
    }
}

```

this가 아닌 object1과 object2 객체를 만들어 this가 아닌 동시에 lock 걸려야 하는 부분을 따로 지정해 줄 수 있다.





**참고**

<https://tourspace.tistory.com/54>

