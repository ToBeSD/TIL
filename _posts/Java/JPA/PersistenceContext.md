# 영속성 컨텍스트

영속성 컨텍스트(Persistence Context)는 JPA에서 엔티티 객체를 관리하는 공간으로, 특정 트랜잭션 범위 내에서 엔티티의 상태를 추적하고 변경을 자동으로 반영하는 역할을 한다.

## 엔티티의 생명주기

엔티티는 아래와같이 4가지의 상태를 가질 수 있다.

비영속 → 영속 → 준영속 → 삭제

### 비영속(Transient) 상태

JPA와 전혀 관련이 없는 상태
new 키워드로 객체를 생성했지만 아직 persist()를 호출하지 않은 상태
DB에 저장되지 않으며, 영속성 컨텍스트와 관계없음

```
Member member = new Member(); // JPA 관리❌
```

### 영속(Persistent) 상태

EntityManager.persist(entity)를 호출하거나,DB에서 엔티티를 조회하여 영속성 컨텍스트에서 관리되는 상태.

```
Member member = new Member();
member.persist(member); // 영속성 컨텍스트의 관리를 받음

Member member = memberRepository.findById(1); // 영속성 컨텍스트의 관리를 받음
```

### 준영속(Detached) 상태

```
entityManager.detach(member);
entityManager.clear();
entityManager.close();
``` 
등을 호출하여 영속성 컨텍스트에서 분리된 상태
JPA가 더 이상 변경을 추적하지 않음
필요 시 `merge()`를 호출하여 다시 영속 상태로 변경 가능

### 삭제(Removed) 상태

`entityManager.remove(entity)`를 호출하면 삭제 상태가 됨
트랜잭션이 `commit`될 때 실제로 `DELETE` 쿼리가 실행됨
JPA 관리 X (DB에서도 삭제됨)

```
entityManager.remove(member); // 삭제 상태
```

## 영속성 컨텍스트의 특징

1. 1차 캐시
2. 변경감지(Dirty Checking)
3. 쓰기지연 
4. 지연로딩(Lazy Loading)

### 1차캐시

DB에서 조회한 엔티티를 영속성 컨텍스트에 저장하여 동일한 트랜잭션 내에서 다시 조회할 경우 DB 조회를 생략하고 캐시된 데이터를 반환한다.

### 영속성 상태에 따른 1차캐시의 동작

#### 영속상태

영속성 컨텍스트의 관리를 받는 엔티티는 Map자료구조에 저장된다.

| Key (식별자) | Entity (객체)                |
|-------------|----------------------------|
| 1           | Member{id=1, name="shim"}  |
| 2           | Member{id=2, name="yosub"} |

영속상태의 엔티티를 한번더 조회하는 작업의 경우 1차캐시에서 해당 엔티티를 가져온다.

```
Optional<Member> updatedMember = memberRepository.findById(1L); // 1차캐시에서 엔티티를 조회
```

#### 준영속상태

영속성 컨텍스트의 관리에서 벗어난 상태.
1차캐시에 저장한 엔티티 정보가 삭제되어 재조회했을때 DB에서 데이터를 가져온다.

```Java
    Optional<Member> member = memberRepository.findById(1L); // member객체가 1차 캐시에 캐시됨
    Member foundMember = member.get();
    foundMember.setName("Dan"); // 1차캐쉬의 member.name = Dan 으로 변경

    entityManager.detach(foundMember); // 영속성 컨텍스트의 관리에서 벗어남 -> 1차캐쉬에서 제거됨

    // 1차캐쉬에 id = 1 인 엔티티가 존재 -> 1차캐쉬에서 엔티티를 조회함
    Optional<Member> updatedMember = memberRepository.findById(1L);

    assertThat(member.get().getName()).isEqualTo("Dan"); // true
    assertThat(updatedMember.get().getName()).isEqualTo("shim"); // true
```

#### 삭제상태

```Java
    Optional<Member> member = memberRepository.findById(1L); // member객체가 1차 캐시에 캐시됨
    Member foundMember = member.get();
    foundMember.setName("Dan"); // 1차캐쉬의 member.name = Dan 으로 변경

    entityManager.remove(foundMember); // 1차캐쉬에서 제거 -> flush() 또는 트랜잭션 종료시 DB에서도 제거가 확정됨

    // 1차캐쉬에 엔티티가 없음 -> DB에서 엔티티를 조회
    Optional<Member> removedMember = memberRepository.findById(1L);
    assertThat(removedMember.isPresent()).isEqualTo(false); // 삭제되었음
```

### 변경 감지(Dirty Checking)

트랜잭션이 끝나기 전, 엔티티의 변경 사항을 감지하고 자동으로 UPDATE 쿼리를 실행하여 반영한다.

### 쓰기 지연

트랜잭션에서 발생한 DB작업이 즉시 DB에 반영되는 것이 아니라, 트랜잭션이 commit될 때 한꺼번에 SQL을 실행하여 성능을 최적화한다.

### 동일성 보장
동일한 영속성 컨텍스트(동일한 트랜잭션) 내에서는 같은 엔티티 객체를 반환하여, == 비교를 해도 true가 나온다.

### 지연로딩(Lazy Loading)

연관된 엔티티를 실제 사용하는 시점까지 로딩을 지연시켜 불필요한 쿼리 실행을 방지한다.