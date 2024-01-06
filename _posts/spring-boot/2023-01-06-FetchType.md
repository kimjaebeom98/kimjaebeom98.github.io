---
layout: single
title: FetchType이란?
categories: spring-boot
tag: [Spring-Boot]
toc: true
---

## FetchType 🤔 ?

<hr>

자바에서 DB와의 상호 작용을 위해 자바 퍼시스턴스 API(JPA)를 사용할 때, `FetchType`은 **엔터티 간의 연관 관계를 정의할 때 사용**한다.

- 엔터티 간의 연관관계 매핑이 되어 있으면(FK) 그 한 엔터티를 조회시 다른 엔터티를 조회하게 되는데, 이때 **다른 엔터티를 어떻게 불러올 것인가**를 설정
- JPA에서는 두 가지 `FetchType`이 있다. **EAGER** OR **LAZY**
- **EAGER** : 연관된 엔터티를 즉시 로딩. 따라서 해당 엔터티가 로드되는 시점에 연관된 엔터티도 함께 로드
  > 예를 들어, 게시판 프로젝트에서 게시물(Board)와 댓글(Reply)가 연관관계가 있다고 할 때 한 게시물 상세보기화면에서 게시물과 댓글 모두 한꺼번에 보여질 때 사용가능
- **LAZY** : 연관된 엔터티는 실제로 사용될 때까지 로딩되지 않음. 따라서 연관된 엔터티가 필요한 시점에 로딩되어 성능을 최적화함.

  > 예를 들어, 게시판 프로젝트에서 게시물(Board)와 댓글(Reply)가 연관관계가 있다고 할 때 한 게시물 상세보기화면에서 게시물은 먼저 불러와지고 댓글을 펼쳐보기 기능을 이용해서 바로 보여질 필요가 없을 때 사용가능

- **JPA FetchType default값**은 `@xxToOne : EAGER`, `@xxToMany : LAZY`

### 1. EAGER (즉시로딩)

<hr>

다음과 같이 Member 엔터티와 Team 엔터티가 N:1 매핑으로 관계를 맺고 있다.

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;
    private String username;

    @ManyToOne(fetch = FetchType.EAGER) //Team을 조회할 때 즉시로딩을 사용하곘다!
    @JoinColumn(name = "team_id")
    Team team;
}

@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;
    private String teamname;
}

```

JPQL로 Member 1건 조회

```java
Member findMember = em.createQuery("select m from Member m", Member.class).getSingleResult();
```

실제 SQL 코드

```sql
//멤버를 조회하는 쿼리
select
    member0_.id as id1_0_,
    member0_.team_id as team_id3_0_,
    member0_.username as username2_0_
from
    Member member0_

//팀을 조회하는 쿼리
select
    team0_.id as id1_3_0_,
    team0_.name as name2_3_0_
from
    Team team0_
where
    team0_.id=?

```

다음과 같이 즉시 로딩(EAGER) 방식을 사용하면 Member를 조회하는 시점에 바로 Team까지 불러오는 쿼리를 날려 한꺼번에 데이터를 불러오는 것을 볼 수 있다.

### 2. EAGER (즉시로딩)

아래와 같이 지연 로딩으로 설정하고 Member를 조회해보면 즉시 로딩 방식과 달리 Team을 조회하는 쿼리가 생성되지 않고 Member를 조회하는 쿼리만 나가고, 실제로 팀을 사용하는 시점에 Team을 조회하는 쿼리가 나간다.

```java

 @ManyToOne(fetch = FetchType.LAZY) //Team을 조회할 때 지연로딩을 사용하곘다!
 @JoinColumn(name = "team_id")
 Team team;
```

JPQL로 Member조회

```java
Member findMember = em.createQuery("select m from Member m", Member.class).getSingleResult();

```

실제 SQL 코드

```sql
//Team을 조회하는 쿼리가 나가지 않음!
select
    member0_.id as id1_0_,
    member0_.team_id as team_id3_0_,
    member0_.username as username2_0_
from
    Member member0_
```

**지연 로딩**을 사용하면 **Member를 조회하는 시점이 아닌 실제 Team을 사용하는 시점에 쿼리가 나가도록 할 수 있다는 장점**이 있다.

위 예제의 즉시 로딩에서는 Member와 연관된 Team이 1개여서 Team을 조회하는 쿼리가 1개 나갔지만, 만약 Member를 조회하는 JPQL을 날렸는데 연관된 Team이 1000개라면? Member를 조회하는 쿼리를 하나 날렸을 뿐인데 Team을 조회하는 SQL 쿼리 1000개가 추가로 나가게 된다.

그렇기 때문에 가급적이면 **기본적으로 지연 로딩을 사용하는 것이 좋다.**
