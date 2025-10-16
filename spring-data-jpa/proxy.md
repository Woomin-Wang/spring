## JPA Proxy

JPA 프록시는 **지연 로딩**(Lazy Loading)을 구현하는 핵심 기술입니다.  
이를 통해 실제 사용되지 않는 연관 엔티티의 조회를 미뤄, 불필요한 데이터베이스 접근을 줄이고 성능을 최적화할 수 있습니다.

<br>
<br>

`em.find()`: **즉시 로딩**

```java
Member member = em.find(Member.class, "member1");
```

- `em.find()`는 엔티티를 실제 사용하든 안 하든, 호출 시점에 즉시 DB를 조회하여 실제 엔티티를 반환합니다.

<br>

`em.getReference()`: **지연 로딩**

```java
Member member = em.getReference(Member.class, "member1");
```

- `em.getReference()`는 실제 엔티티를 사용하는 시점까지 DB 조회를 미룹니다. 
- 호출 시점에는 실제 엔티티가 아닌, DB 접근을 위임하는 **프록시(Proxy) 객체**를 반환합니다.


<br>

## 프록시 구조와 동작 원리

<img width="500" height="200" alt="image" src="https://github.com/user-attachments/assets/7c327c8c-88cd-4029-8f7f-f87b1508e072" />

<img width="500" height="200" alt="image" src="https://github.com/user-attachments/assets/ec522461-095f-42b6-b6bf-c3eae3f7b1d8" />

프록시 클래스는 실제 클래스를 상속받아 만들어지므로, 겉모습은 실제 클래스와 같습니다.  
내부적으로는 실제 객체에 대한 참조(`target`)를 보관하며, 프록시 객체의 메서드가 호출되면 이 `target`이 가리키는 실제 객체의 메서드를 대신 호출해 줍니다.

<br>

### 프록시 초기화

<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/3b544e35-955b-447e-b851-83699fe203ad" />

- 프록시 객체는 `member.getName()`처럼 실제 값을 사용하는 시점에 DB를 조회해서 실제 엔티티 객체를 생성하고,
- 이 객체를 영속성 컨텍스트에 로딩하는데 이를 **프록시 초기화**라고 합니다.
- 즉, 프록시가 초기화되었다는 것은 내부의 `target` 참조가 더 이상 `null`이 아니라, **참조할 실제 엔티티가 영속성 컨텍스트에 로딩**되었음을 의미합니다.

<br>
<br>

### 프록시의 주요 특징

1. 프록시 객체는 처음 사용할 때 **한 번만 초기화**됩니다.
2. 프록시 객체를 초기화할 때, 프록시 객체가 실제 엔티티로 바뀌는 것이 아니라 프록시 객체를 통해서 **실제 엔티티에 접근이 가능**합니다.
3. 프록시 객체는 원본 엔티티를 상속받으므로, 타입 체크 시 `==` 비교 대신 `instanceof`를 사용해야 합니다.
4. 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 `em.getReference()`를 호출해도 **실제 엔티티를 반환**합니다.
5. 영속성 컨텍스트의 도움을 받을 수 없는 **준영속** 상태일 때, 프록시를 초기화하면 예외가 발생한다.

<br>
<br>

## 영속성 컨텍스트와 프록시 동작

JPA는 하나의 트랜잭션 안에서, 같은 PK 값으로 조회된 엔티티는 항상 동일한 메모리 주소의 인스턴스여야 한다는 **동일성 보장** 원칙을 가집니다.

<br>

**em.getReference()의 동작 순서**  

>`em.getReference()`는 단순히 프록시를 즉시 생성하는 것이 아니라, 영속성 컨텍스트의 상태에 따라 다르게 동작합니다.  

1. 요청한 ID에 해당하는 엔티티가 1차 캐시에 이미 존재하면 (Cache Hit), DB에 접근할 필요 없이 **실제 엔티티 참조를 즉시 반환**합니다.

2. 1차 캐시에 없다면 (Cache Miss), DB에 `SELECT` 쿼리를 보내지 않고 ID 값만 가진 **프록시 객체**를 생성하여 반환합니다.

<br>
<br>


✅ **예제1: 1차 캐시에 실제 엔티티가 이미 있는 경우**

```java
Member member = new Member();
member.setName("홍길동");
em.persist(member);

Member findMember = em.find(Member.class, member.getId());
System.out.println("findMember.getClass() = " + findMember.getClass()); // class hellojpa.domain.Member

Member proxyMember = em.getReference(Member.class, member.getId());
System.out.println("proxyMember.getClass() = " + proxyMember.getClass()); // class hellojpa.domain.Member (프록시가 아님)

System.out.println("proxyMember == findMember: " + (proxyMember == findMember)); // true
```

- `em.find()`로 실제 엔티티를 먼저 조회하면, 해당 엔티티는 1차 캐시에 저장됩니다.
- 이후 `em.getReference()`를 호출하면, 동일성 보장 원칙에 따라 프록시가 아닌 1차 캐시에 있던 실제 엔티티를 반환합니다.

<br>
<br>

✅ **예제2: 1차 캐시에 프록시가 이미 있는 경우****

```java
Member member = new Member();
member.setName("홍길동");
em.persist(member);

em.flush();
em.clear();

Member proxyMember = em.getReference(Member.class, member.getId());
System.out.println("proxyMember.getClass() = " + proxyMember.getClass()); // class ...$HibernateProxy...

Member findMember = em.find(Member.class, member.getId());
System.out.println("findMember.getClass() = " + findMember.getClass()); // class ...$HibernateProxy...

System.out.println("proxyMember == findMember: " + (proxyMember == findMember)); // true
```

- 반대로 `em.getReference()`를 통해 프록시가 1차 캐시에 먼저 등록된 경우, 이후에 `em.find()`를 호출하면 동일성 보장 원칙에 따라 **프록시 객체를 그대로 반환**합니다.
- 하지만 `em.find()`는 '반드시 실제 값을 반환한다'는 약속이 있으므로, 이 약속을 지키기 위해 JPA는 반환하려는 **프록시 객체를 강제로 초기화**합니다. 이 과정에서 `SELECT` 쿼리가 실행됩니다.

<br>
<br>

### 준영속 상태와 프록시

영속성 컨텍스트의 도움을 받을 수 없는 **준영속 상태**일 때, 초기화되지 않은 프록시를 사용하려고 하면 예외가 발생합니다.  
(하이버네이트는 `org.hibernate.LazyInitializationException` 예외를 발생시킴)

```java
Member member = new Member();
member.setName("홍길동");
em.persist(member);

em.flush();
em.clear();

Member proxyMember = em.getReference(Member.class, member.getId());

em.detach(proxyMember); // 프록시를 영속성 컨텍스트에서 분리 (준영속)
//em.close();           // 영속성 컨텍스트를 닫아도 동일

// LazyInitializationException 예외 발생!
System.out.println("proxyMember.getName() = " + proxyMember.getName());
```

- `proxyMember`가 준영속 상태가 되었고, `proxyMember.getName()` 호출 시 프록시 초기화가 이루어져야 하는데, 초기화를 도와줄 영속성 컨텍스트와의 연결이 끊겼기 때문에 예외가 발생한 경우입니다.



<br>
<br>
