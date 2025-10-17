# 상속 관계 매핑 (Inheritance Mapping)

> 객체지향의 **상속** 개념을 관계형 DB의 **슈퍼타입-서브타입** 모델과 매핑하는 기술입니다. JPA에서는 상속 구조를 DB 테이블로 구현하기 위해 세 가지 전략을 제공합니다.

<br>

### 1. 조인 전략 (Joined Strategy)

> 부모와 자식 테이블을 모두 생성하고, 자식 테이블이 부모의 PK를 자신의 PK이자 FK로 참조하는 방식입니다.

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/e9d1466d-6585-45e8-9367-1631d0c127da" />

```java
// 부모
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Item {
    @Id @GeneratedValue
    private Long id;
    // ... 공통 필드 ...
}

// 자식
@Entity
public class Book extends Item {
    // ... Book 고유 필드 ...
}
```

**장점:**
- **정규화**: 테이블 구조가 깔끔하고 정규화되어 있습니다.
- **공간 효율성**: 데이터가 필요한 컬럼에만 저장되어 공간 낭비가 없습니다.
- **참조 무결성**: 외래 키 제약조건을 활용할 수 있습니다.

**단점:**
- **성능**: 조회 시 항상 **조인**(JOIN)이 발생하여 성능이 저하될 수 있습니다.
- **복잡성**: INSERT 시 부모와 자식 테이블에 각각 쿼리가 실행됩니다.

<br>
<br>

### 2. 싱글 테이블 전략 (Single Table Strategy)

> 하나의 테이블에 부모와 모든 자식의 데이터를 통합하고, 구분 컬럼(`@DiscrimintorColumn`을 사용하여 타입을 식별하는 방식입니다.

<img width="200" height="300" alt="image" src="https://github.com/user-attachments/assets/a46bd2d3-22b0-430a-b5a4-1c3be5cffe1d" />

```java
// 부모
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "ITEM_TYPE")
public abstract class Item {
    @Id @GeneratedValue
    private Long id;
    // ... 공통 필드 ...
}

// 자식
@Entity
@DiscriminatorValue("BOOK")
public class Book extends Item {
    // ... Book 고유 필드 ...
}
```

**장점:**
- **성능**: 조인이 필요 없어 조회 성능이 가장 빠릅니다.
- **단순성**: 조회 쿼리가 단순합니다.

**단점:**
- **공간 낭비**: 자식 엔티티의 고유 컬럼들은 다른 타입의 데이터가 저장될 때 NULL로 채워져 공간을 낭비할 수 있습니다.
- **무결성**: 모든 자식 컬럼이 NULL을 허용해야 하므로 NOT NULL 제약조건을 걸기 어렵습니다.

> 이 전략에서는 타입을 구분할 방법이 반드시 필요하므로 `@DiscriminatorColumn`이 필수입니다. 

<br>
<br>

### 3. 구현 테이블마다 테이블 전략 (Table Per Class Strategy)

> 자식 엔티티마다 독립적인 테이블을 생성하며, 부모의 공통 속성까지 모두 포함하는 방식입니다. 

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/711eb3be-5bf0-402a-8f57-14252f1c5677" />


```java
// 부모
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {
    @Id @GeneratedValue
    private Long id;
    // ... 공통 필드 ...
}

// 자식
@Entity
public class Book extends Item {
    // ... Book 고유 필드 ...
}
```

> 데이터베이스 설계자와 ORM 전문가 둘 다 권장하지 않는 전략입니다.

<br>
<br>

## @MappedSuperclass

> `@MappedSuperclass`는 상속 관계 매핑이 아닙니다. 부모 클래스에 정의된 **매핑 정보(필드)를 자식 클래스에게 그대로 물려주는 역할을 합니다.**

```java
@MappedSuperclass
public abstract class BaseEntity {
    @Id @GeneratedValue
    private Long id;
    private String name;
    // ...
}

@Entity
public class Member extends BaseEntity { // id, name 필드를 상속받음
    private String email;
    // ...
}
```

- **엔티티가 아님**: 테이블과 직접 매핑되지 않으며, `em.find()` 등으로 조회할 수 없습니다.
- **정보 상속용**: 자식 클래스에 필드 매핑 정보만 제공합니다.
- **추상 클래스 권장**: 직접 생성해서 사용할 일이 없으므로, 보통 **추상 클래스(abstract class)로 만듭니다.**


<br>
<br>
