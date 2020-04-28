# try with resources를 사용해라

Effective Java를 보다가 신기한 부분을 발견했다.



나는 보통 try ~ catch, try ~ catch ~ finally 등 단순히 예외처리를 할 때 사용한다. 라는게 내가 아는 전부였다.





자바 라이브러리에는 close메소드를 호출해 직접 닫아줘야 하는 자원이 많다.

대표적으로 InputStream, OutputStream, java.sql.Connection 등이 있다.



대부분이 try-finally를 사용해 close메소드를 사용하곤 한다.

```java
static String firstLineOfFile(String path)throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try{
        return br.readLine();
    }finally{
        br.close()
    }
}
```

 하지만 자원을 하나 더 사용한다면?

```java
static void copy(String src, String dst) throws IOException{
    InputStream in = new FileInputstream(src);
    try{
        OutputStream out = new FileOutputStrea(dst);
        try{
            byte[] buf = new byte[BUFFER_SIZE];
            int n
            while((n = in.read(buf)) >= 0){
                out.write(buf, 0, n);
            }finally{
                out.close();
            }
        }finally{
            in.close();
        }
    }
}
```

코드가 너무 복잡해진다.



자바 7에서는 AutoCloseable 인터페이스를 구현해 해결했다.

우리가 자주 사용하는 라이브러리에 있는 자원 클래스들은 대부분 AutoCloseable 인터페이스를 이미 구현하고 있다고 한다.

아래와 같이 단순히 void를 반환하는 close 메소드만 정의되어 있다.

```java
public interface AutoCloseable { 
	void close() throws Exception; 
}
```





try-finally로 작성된 코드를 try-with-resources로 변경해보자.

```java
public static String firstLineOfFile(String path) throws IOException{
    try(BufferedReader br = new BufferedReader(new FileReader(path))){
        return br.readLine();
    }catch(Exception e){
        return "";
    }
}
```

try-finally보다 코드가 간편해졌다. 또한 try 블록이 끝나면 자원은 자동으로 해제되어 직접 close하지 않아도 된다.



여러개의 자원을 생성하는 경우도 마찬가지로 간단해진다.

```java
try(InputStream in = new FileInputstream(src);
	  OutputStream out = new FileOutputStrea(dst)){
          
	  }
```

이런식으로 자원 객체 생성부분을 try 안에 넣어준다.





try-catch-resources를 사용하기 위해서는 해당 리소스 객체가 java.lang.AutoCloseable 인터페이스를 구현하고 있어야 한다. 아래와 같이 API 도큐먼트에서 AutoCloseable 인터페이스를 찾아 보면 “All Known Implementing Classes:”에서 찾아볼 수 있다.