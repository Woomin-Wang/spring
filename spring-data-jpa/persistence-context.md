## EntityManagerFactory & EntityManager

<img width="600" height="400" alt="image" src="https://github.com/user-attachments/assets/ba4d9321-e349-43f4-a6de-bfa0ebed1349" />

**EntityManagerFactory**

- `EntityManager`를 생성하는 공장입니다.
- 생성 비용이 매우 비싸므로, 애플리케이션 전체에서 **단 하나만 생성해서 공유**하는 것이 원칙입니다.
- 여러 스레드가 동시에 접근해도 안전합니다. **(Thread-Safe)**

**EntityManager**

- 영속성 컨텍스트를 통해 엔티티를 CRUD하는 실제 일꾼입니다.
- 생성 비용이 거의 들지 않으며, 요청마다 생성하고 폐기합니다.
- 여러 스레드가 동시에 접근하면 동시성 문제가 발생하므로, **스레드 간에 절대 공유하면 안 됩니다.**
- DB 커넥션은 트랜잭션을 시작하는 등 꼭 필요한 시점에만 획득하여 사용합니다.

<br>
<br>

# 영속성 컨텍스트 (Persistence Context)

> 엔티티를 영구 저장하는 환경이라는 뜻으로, `EntityManager`가 엔티티를 보관하고 관리하는 **논리적인 작업 공간**입니다.

<br>

**엔티티의 생명주기 (Lifecycle)**

<img width="600" height="400" alt="image" src="https://github.com/user-attachments/assets/3ece86c9-2207-4010-aa56-f14bfd5ecbaa" />

- **비영속 (New)**: 순수한 new 객체 상태. 영속성 컨텍스트와 아무 관계가 없습니다.
- **영속 (Managed)**: `em.persist()` 등을 통해 **영속성 컨텍스트에 의해 관리되는 상태**입니다.
- **준영속 (Detached)**: 영속성 컨텍스트에 저장되었다가 분리된 상태입니다.
- **삭제 (Removed)**: `em.remove()`를 통해 삭제된 상태입니다.

<br>

## 영속성 컨텍스트의 특징 

> 영속성 컨텍스트가 엔티티를 관리하기 때문에 다음과 같은 강력한 이점을 얻을 수 있습니다.

<br>

### 1차 캐시

> 영속성 컨텍스트는 내부에 **1차 캐시**라는 공간을 가집니다.

<img width="400" height="250" alt="image" src="https://github.com/user-attachments/assets/225f491c-71c5-48a7-a05b-d56ca857b9b0" />

<img width="460" height="220" alt="image" src="https://github.com/user-attachments/assets/7757e880-5496-41f9-9396-b3893c480619" />

- `em.persist(member)`: 엔티티를 1차 캐시에 저장합니다.
- `em.find(Member.class, "id")`:
  - **먼저 1차 캐시**에서 해당 ID의 엔티티를 찾습니다.
  - **있으면** DB를 거치지 않고 캐시의 엔티티를 즉시 반환합니다.
  - **없으면** DB에서 조회한 후, **1차 캐시에 저장하고** 엔티티를 반환합니다.

<br>

### 동일성(Identity) 보장

> 1차 캐시 덕분에, 같은 트랜잭션 내에서 같은 ID로 조회한 엔티티는 **항상 같은 메모리 주소의 인스턴스임이 보장**됩니다.

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");
System.out.println(a == b); // true
```

<br>

### 트랜잭션을 지원하는 쓰기 지연 (Transactional Write-Behind)

> `em.persist()`를 호출해도 `INSERT` SQL이 바로 DB에 전송되지 않습니다.

```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
//엔티티 매니저는 데이터 변경 시 트랜잭션을 시작해야 한다.

transaction.begin(); //[트랜잭션] 시작
em.persist(memberA);
em.persist(memberB);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

//커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
transaction.commit(); //[트랜잭션] 커밋
```

<br>

<img width="400" height="250" alt="image" src="https://github.com/user-attachments/assets/4e9f43bb-5513-45a6-9d01-b9c013c57d88" />

<img width="450" height="230" alt="image" src="https://github.com/user-attachments/assets/d2e1b900-ef06-4cc3-a8b9-cf128233223a" />


1. `em.persist()` 호출 시, `INSERT` SQL을 생성하여 **쓰기 지연 SQL 저장소**에 쌓아둡니다.
2. `transaction.commit()`이 호출될 때, 저장소에 모아뒀던 SQL들을 **한 번에 DB로 플러시** 합니다.

<br>

### 변경 감지 (Dirty Checking)

> `em.update()` 같은 메서드 없이, **영속 상태의 엔티티 데이터를 수정하기만 하면 자동으로 DB에 반영**됩니다.

<img width="600" height="400" alt="image" src="https://github.com/user-attachments/assets/5b100e11-8bd1-4fe5-924d-9abb2863512a" />

**동작 원리**

1. **스냅샷 생성**: JPA는 엔티티를 1차 캐시에 저장할 때, 최초 상태를 복사한 스냅샷을 함께 보관합니다.
2. **`commit` 시점**: 트랜잭션 커밋 시 `flush`가 호출됩니다.
3. **변경 감지**: `flush` 과정에서, 영속성 컨텍스트의 모든 엔티티를 **현재 상태와 스냅샷을 비교**합니다.
4. **UPDATE SQL 생성**: 변경된 엔티티가 발견되면, `UPDATE` SQL을 생성하여 쓰기 지연 SQL 저장소에 보냅니다.
   - 기본: 성능 최적화를 위해 **모든 필드를 업데이트**하는 쿼리를 생성합니다. (쿼리 캐싱 재사용)
   - 동적: `@DynamicUpdate` 어노테이션을 붙이면, **변경된 필드**만으로 `UPDATE` 쿼리를 동적으로 생성합니다.
6. **플러시**: 저장소의 SQL들을 DB에 전송합니다.

<br>

### Flush

> 플러시(Flush)는 영속성 컨텍스트의 **쓰기 지연 SQL 저장소**에 쌓여있는 변경 내용(`INSERT`, `UPDATE`, `DELETE` 쿼리들)을 DB에 전송하여 동기화하는 작업입니다.

<br>

**플러시가 발생하는 경우**

- `em.flush()` **직접 호출**:
  - 개발자가 코드로 직접 `flush()`를 호출하여 DB와 동기화 시점을 제어할 수 있습니다.

- **트랜잭션 커밋 시 자동 호출**
  - `tx.commit()`을 호출하면, JPA는 트랜잭션을 커밋하기 전에 내부적으로 `flush()`를 먼저 호출하여 변경 내용을 DB에 반영합니다.  

- **JPQL 쿼리 실행 시 자동 호출**
   - JPQL은 DB에서 직접 데이터를 조회합니다.
   - 만약 영속성 컨텍스트에만 있고 아직 DB에 반영되지 않은 데이터가 있다면, JPQL 쿼리 결과에서 누락될 수 있습니다.
   - 이 데이터 불일치를 막기 위해 JPQL는 실행 전에 플러시가 먼저 동작합니다.

```java
// 1. memberA, B, C는 1차 캐시와 쓰기 지연 SQL 저장소에만 존재 🛒
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);

// 2. 이 시점에 JPQL 쿼리가 실행되면 memberA, B, C는 조회되지 않음
//    따라서 JPA는 이 쿼리가 실행되기 직전에 flush()를 자동으로 호출함!
List<Member> members = em.createQuery("select m from Member m", Member.class)
                         .getResultList();
```

<br>
<br>




