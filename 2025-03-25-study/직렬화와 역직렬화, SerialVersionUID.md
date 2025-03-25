![image](https://github.com/user-attachments/assets/e21f9091-b319-4c9e-8c45-949004594c72)



# 직렬화와 역직렬화의 필요성

객체 직렬화(Serialization)와 역직렬화(Deserialization)는 Java 애플리케이션에서 왜 중요한 역할을 하는 메커니즘으로 작동할까?

왜 Spring Framework에서는 클래스 내에 `SerialVersionUID`를 만들도록 권고할까?

<br />
<br />

## 🧑‍🏭 객체 직렬화와 역직렬화

> **객체 직렬화(Serialization)** 는 Java 객체를 바이트 스트림으로 변환하는 과정. 즉, 메모리에 있는 객체의 상태를 영속적인 형태로 변환

> **역직렬화(Deserialization)** 는 _바이트 스트림_ 을 다시 Java 객체로 복원하는 과정

<br />
<br />

## 📌 직렬화와 역직렬화가 필요한 이유

### 1. 데이터 영속성(Persistence)
- **객체 상태 저장**: 애플리케이션이 종료되어도 객체의 상태를 파일 시스템에 저장하여 나중에 복원 가능
  
> 💡**why?**
> 
> 메모리 내 객체는 가상 메모리 주소, 포인터, 참조 등의 복잡한 구조를 가지고 있어 그대로 저장할 수 없음. 직렬화는 이러한 구조를 파일 시스템이나 데이터베이스에 저장할 수 있는 연속적인 바이트 형태로 변환.
>
> 직렬화는 객체 간의 참조 관계를 포함한 전체 객체 그래프를 보존. 이는 단순히 필드 값만 저장하는 것보다 객체의 전체 상태를 정확히 캡처

<br />
<br />

- **데이터베이스 저장**: 복잡한 객체 그래프를 데이터베이스에 저장할 때 사용
> 💡**why?**
>
> 관계형 데이터베이스는 복잡한 객체 구조를 직접 표현하기 어려움. 직렬화된 객체는 **BLOB(Binary Large Object)** 타입으로 단일 컬럼에 저장될 수 있음.
> 
> **ORM 매핑 회피** : 복잡한 객체 계층을 테이블 관계로 매핑하는 과정을 생략할 수 있어 개발 복잡성을 줄일 수 있음.

<br />
<br />

### 2. 네트워크 통신
- **원격 메서드 호출(RMI)**: 분산 시스템에서 객체를 네트워크로 전송할 때 필요
  
> 💡**why?**
> 
> **전송 가능한 형태** : 객체는 메모리 내 표현이 시스템에 종속적이므로 네트워크로 직접 전송할 수 없음. 직렬화는 이를 전송 가능한 바이트 스트림으로 변환
> 
> **네트워크 프로토콜 호환성** : 네트워크 프로토콜은 일반적으로 바이트 스트림을 기반으로 설계되어 있음. 직렬화된 객체는 이러한 프로토콜과 자연스럽게 호환

<br />
<br />

- **웹 서비스**: REST API나 SOAP 같은 웹 서비스에서 데이터 교환을 위해 사용
- **소켓 통신**: 네트워크를 통해 객체를 전송할 때 직렬화가 필요
  
> 💡**why?**
> 
> **프로토콜 독립성:**  직렬화된 데이터는 HTTP, TCP, UDP 등 다양한 전송 프로토콜로 쉽게 전송
> 
> **언어 간 호환성:**  JSON이나 XML 같은 표준 형식으로 직렬화하면 다른 프로그래밍 언어로 작성된 시스템과도 통신

<br />
<br />


### 3. 캐싱(Caching)
- **메모리 외부 캐싱**: Redis나 Memcached 같은 캐시 시스템에 객체를 저장할 때 직렬화가 필요
  
> 💡**why?**
> 
> 저장 공간 최적화: 메모리는 비용이 높은 자원이므로, 자주 사용되지 않는 객체를 직렬화하여 디스크에 저장함으로써 메모리 사용을 최적화 가능
> 
> 캐시 키-값 구조: Redis나 Memcached 같은 캐시 시스템은 바이트 배열 형태의 값을 저장하는 데 최적화되어 있습니다. 직렬화는 객체를 이러한 형태로 변환

<br />
<br />

- **디스크 기반 캐싱**: 계산 비용이 큰 객체를 디스크에 캐싱할 때 사용

> 💡**why?**
> 
> IO 스트림 호환성: 파일 시스템 I/O는 바이트 스트림 기반으로 동작합니다. 직렬화된 객체는 이러한 I/O 작업에 자연스럽게 적합합니다.
> 
> 재구성 용이성: 객체의 상태를 그대로 보존하여 디스크에 저장하면, 나중에 원본과 동일한 상태로 객체를 재구성할 수 있습니다.


<br />
<br />

### 4. 세션 관리
- **HTTP 세션**: 웹 애플리케이션에서 사용자 세션 정보를 저장하고 로드할 때 사용
  
> 💡**why?**
> 
> 서버 자원 관리: 웹 서버는 수많은 동시 사용자를 처리해야 하므로, 모든 세션 객체를 메모리에 유지하는 것은 비효율적. 직렬화를 통해 사용하지 않는 세션을 디스크로 옮길 수 있음(세션 스와핑).
>
> 서버 재시작 대응: 직렬화된 세션은 서버 재시작 시에도 복원될 수 있어 사용자 경험이 끊기지 않음

<br />
<br />

- **분산 세션**: 여러 서버에 걸쳐 세션을 공유할 때 직렬화가 필요

> 💡**why?**
> 
> 서버 간 상태 공유: 로드 밸런싱 환경에서 사용자 요청이 여러 서버로 분산될 수 있음. 직렬화된 세션 데이터는 서버 간에 쉽게 공유 가능.
>
> 중앙 저장소 활용: 직렬화된 세션 데이터는 Redis와 같은 중앙 저장소에 저장되어 모든 서버에서 접근 가능.


<br />
<br />


### 5. 딥 복사(Deep Copy)
- 복잡한 객체 구조의 완전한 복사본을 만들 때 직렬화와 역직렬화를 활용 가능
> 💡**why?**
> 
> **참조 독립성**: 직렬화와 역직렬화를 통한 복사는 원본 객체와 완전히 독립된 새 객체를 생성. 이는 모든 중첩 객체와 참조까지 복사.
> 
> **구현 단순성**: 복잡한 재귀적 복사 로직을 직접 구현하는 대신, 객체 직렬화 메커니즘을 활용하여 간단하게 딥 복사를 수행

<br />
<br />
<br />

## 📌 실제 사용 예시

```java
// 객체 직렬화 예시
public void saveObject(User user, String filename) throws IOException {
    try (FileOutputStream fileOut = new FileOutputStream(filename);
         ObjectOutputStream out = new ObjectOutputStream(fileOut)) {
        out.writeObject(user); // 객체를 직렬화하여 파일에 저장
    }
}

// 객체 역직렬화 예시
public User loadObject(String filename) throws IOException, ClassNotFoundException {
    try (FileInputStream fileIn = new FileInputStream(filename);
         ObjectInputStream in = new ObjectInputStream(fileIn)) {
        return (User) in.readObject(); // 파일에서 읽어 객체로 역직렬화
    }
}
```

<br />
<br />

## 🪪 SerialVersionUID란?

```java
public class User implements Serializable {
    private static final long serialVersionUID = 1L; // 명시적 선언
    // 클래스 필드와 메서드
}
```

**SerialVersionUID란?**
- 직렬화된 객체와 클래스 간의 호환성을 확인하는 데 사용되는 고유 식별자
- 클래스 버전 관리를 위한 해시값으로 볼 수 있음.

<br />
<br />

**명시적으로 선언하는 이유:**
1. **버전 관리**: 클래스가 변경되더라도 호환성을 유지 가능.
2. **예측 가능성**: 자동 생성되는 UID는 컴파일러 구현에 따라 달라질 수 있어 예측이 어려움.
3. **안정성**: 명시적 선언으로 클래스 변경 시에도 같은 UID를 유지할 수 있음.



<br />

### SerialVersionUID가 다를 때 발생하는 문제

SerialVersionUID가 다를 경우 다음과 같은 문제가 발생

1. **InvalidClassException 발생**: 역직렬화 시 `java.io.InvalidClassException`이 발생
2. **데이터 접근 불가**: 저장된 직렬화 데이터를 더 이상 사용할 수 없게 됨
3. **오류 메시지**: "local class incompatible: stream classdesc serialVersionUID = X, local class serialVersionUID = Y"와 같은 오류가 발생

이는 Java가 클래스의 구조가 변경되었다고 판단하여 데이터 무결성을 보호하기 위해 역직렬화를 거부하는 것!

<br />

### 클래스 필드 변경 시 역직렬화 유지 방법

클래스의 필드가 변경되었을 때도 역직렬화가 가능하도록 하는 방법:

1. **SerialVersionUID 유지**: 클래스가 변경되더라도 SerialVersionUID를 동일하게 유지

<br />

2. **readObject/writeObject 커스터마이징**:
   ```java
   private void writeObject(ObjectOutputStream out) throws IOException {
       out.defaultWriteObject(); // 기본 직렬화 수행
       // 추가 데이터 직렬화
   }
   
   private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
       in.defaultReadObject(); // 기본 역직렬화 수행
       // 추가 데이터 역직렬화 또는 기본값 설정
   }
   ```

<br />

3. **transient 키워드 사용**: 직렬화에서 제외할 필드에 사용
   ```java
   private transient int tempValue; // 직렬화에서 제외
   ```

<br />


4. **Externalizable 인터페이스 구현**: Serializable보다 더 세밀한 제어가 가능
   ```java
   public class User implements Externalizable {
       @Override
       public void writeExternal(ObjectOutput out) throws IOException {
           // 필요한 필드만 직접 직렬화
       }
       
       @Override
       public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
           // 커스텀 역직렬화 로직
       }
   }
   ```

<br />
<br />


## 🥲 Java 객체 직렬화와 역직렬화 관련 문제

### ClassNotFoundException 발생 이유

객체를 직렬화한 후 역직렬화할 때 클래스의 classpath가 변경되면 ClassNotFoundException이 발생.

1. **클래스 로딩 메커니즘**: 역직렬화 과정에서 JVM은 직렬화된 객체의 클래스를 현재 classpath에서 찾아 로드해야 함
2. **FQCN(Fully Qualified Class Name) 사용**: 직렬화 시 객체의 완전한 클래스 이름(패키지 포함)이 저장
3. **클래스 위치 변경**: 클래스의 패키지 경로가 변경되면, 역직렬화 시 JVM은 원래 경로에서 클래스를 찾을 수 없게 됨

따라서 `com.example.User`가 `com.company.User`로 변경되었다면, 원래 경로인 `com.example.User`를 찾을 수 없어 ClassNotFoundException이 발생

<br />
<br />

### 직렬화의 대안

Java 직렬화에는 보안 및 유연성 측면에서 한계가 있어, 다음과 같은 대안이 자주 사용.

- **JSON/XML 변환**: Jackson, GSON, JAXB 등의 라이브러리를 사용
- **Protocol Buffers**: Google에서 개발한 구조화된 데이터 직렬화 방식
- **MessagePack**: JSON보다 효율적인 바이너리 직렬화 형식
- **Kryo**: 고성능 Java 직렬화 라이브러리

이러한 직렬화와 역직렬화 메커니즘은 현대 애플리케이션 개발에서 데이터 교환과 저장의 핵심 요소로 사용.



