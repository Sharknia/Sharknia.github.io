---
IDX: "NUM-215"
tags:
  - Spring
  - DataBase
  - Kotlin
  - ORM
description: "Spring의 ORM을 사용하자"
series: "Kotlin과 Spring을 활용한 프로젝트"
update: "2024-09-05T06:45:00.000Z"
date: "2024-09-05"
상태: "Ready"
title: "JPA(Java Persistence API) (1)"
---
![](image1.png)
## ORM(Object-Relational Mapping)

생각해보니 ORM을 사용한 적이 많았는데, 이게 어떤 것인지 명확하게 짚고 넘어갔던 적은 없던 것 같습니다. Spring의 ORM을 써보려고 하니, 이번 기회에 한 번 정리하고 넘어가려고 합니다. 

### ORM의 정의

ORM은 객체 지향 프로그래밍(OOP) 언어에서 사용하는 객체와 관계형 데이터베이스의 테이블을 자동으로 매핑해주는 기법입니다. 이를 통해 데이터베이스와의 상호작용을 SQL이 아닌 객체를 통해 처리할 수 있습니다. ORM을 사용하면 데이터베이스의 테이블을 객체로 추상화하여 비즈니스 로직을 더 직관적으로 표현할 수 있으며, 코드의 재사용성과 유지보수성을 높일 수 있습니다.

### 객체 지향 프로그래밍과 관계형 데이터베이스의 차이 

- 객체 지향 프로그래밍(OOP)에서는 데이터를 객체로 모델링하고, 이 객체들 간의 상호작용을 통해 프로그램을 구성합니다. 객체는 속성(필드)과 행동(메서드)을 가집니다.

- 관계형 데이터베이스(RDB)에서는 데이터를 테이블로 구조화하여 저장합니다. 테이블은 행과 열로 이루어져 있으며, 관계를 통해 데이터를 연결합니다.

- 주요 차이점은 객체는 메모리에서 동작하는 반면, 데이터베이스는 저장 공간에 데이터를 영구적으로 저장한다는 점입니다. 객체 간의 관계는 참조로 연결되지만, 테이블 간의 관계는 외래 키(Foreign Key)로 연결됩니다. 또한 객체는 다형성(polymorphism)을 지원하지만, 테이블은 그렇지 않습니다.

    
        다형성(polymorphism)은 객체 지향 프로그래밍(OOP)에서 중요한 개념 중 하나로, 같은 메서드나 연산자가 여러 가지 형태로 동작할 수 있는 성질을 말합니다. 이를 통해 코드의 유연성과 확장성을 높일 수 있습니다.

### 장점과 단점

#### 장점

- 생산성 향상: SQL을 직접 작성하지 않고도 데이터베이스 작업을 할 수 있으므로 개발자가 비즈니스 로직에 집중할 수 있습니다. 복잡한 SQL 쿼리를 줄이고, 간결한 코드로 데이터 접근 로직을 처리할 수 있습니다.

- 유지보수성: 객체 지향 프로그래밍의 구조를 따르기 때문에 비즈니스 로직과 데이터베이스 로직을 분리할 수 있어 코드의 재사용성 및 유지보수성이 높아집니다.

- 데이터베이스 독립성: ORM은 데이터베이스에 종속되지 않으며, 동일한 코드를 사용해 여러 데이터베이스와 상호작용할 수 있습니다. 즉, 데이터베이스가 변경되어도 코드 수정이 최소화됩니다.

- 자동화된 매핑: 객체와 테이블의 매핑이 자동으로 이루어지기 때문에 데이터베이스의 스키마를 수동으로 매핑하는 부담이 줄어듭니다.

#### 단점

- 성능 저하: 복잡한 쿼리나 대량의 데이터를 다룰 때는 ORM이 생성하는 쿼리가 비효율적일 수 있습니다. 특히, [N+1 문제](https://sharknia.github.io/N1-문제)와 같은 성능 이슈가 발생할 수 있으며, 이를 해결하기 위해서는 추가적인 최적화가 필요합니다.

- 추상화의 복잡성: ORM은 데이터베이스와의 상호작용을 추상화하지만, 너무 많은 추상화는 특정한 데이터베이스 특성을 활용하지 못하게 하거나 디버깅을 어렵게 만들 수 있습니다.

- 복잡한 쿼리의 한계: ORM은 단순한 CRUD 작업에는 효율적이지만, 복잡한 쿼리나 고성능이 요구되는 작업에서는 SQL이 더 유리할 수 있습니다. 이때는 Native Query를 사용해야 할 수 있습니다.

### ORM을 사용하지 않았을 때의 문제점

- 반복되는 코드: SQL 쿼리 작성과 결과 매핑을 직접 처리해야 하기 때문에 비슷한 패턴의 코드가 반복적으로 나타날 수 있습니다. 이는 코드의 양을 증가시키고 유지보수를 어렵게 만듭니다.

- 비즈니스 로직과 데이터베이스 로직의 혼합: SQL 쿼리가 비즈니스 로직에 직접 포함될 경우, 코드의 가독성과 유지보수성이 저하됩니다. 이는 복잡한 프로젝트에서 특히 문제로 작용할 수 있습니다.

- 데이터베이스 종속성: 특정 데이터베이스에 종속된 SQL 쿼리를 사용하면, 데이터베이스가 변경될 때마다 코드를 수정해야 할 필요가 생깁니다. ORM을 사용하면 데이터베이스와 독립적으로 애플리케이션을 설계할 수 있습니다.

- 수동 매핑의 불편함: 데이터베이스에서 가져온 데이터를 객체에 수동으로 매핑하는 과정에서 발생하는 실수나 비효율성이 존재할 수 있으며, 이를 관리하는 데 많은 시간이 소모될 수 있습니다.

## JPA

### JPA란? 

- JPA(Java Persistence API)는 자바에서 객체 지향 프로그래밍을 기반으로 관계형 데이터베이스를 다룰 수 있게 해주는 표준 인터페이스입니다. 즉, 객체와 데이터베이스 테이블 간의 매핑을 제공하는 프레임워크 표준입니다.

- JPA를 사용하면 SQL을 직접 작성하는 대신, 객체를 통해 데이터베이스의 데이터를 조회, 삽입, 수정, 삭제하는 작업을 수행할 수 있습니다.

- JPA는 데이터베이스와의 상호작용을 추상화하여 데이터베이스 종속성을 줄이고, 객체 지향적인 방식으로 애플리케이션을 설계할 수 있게 도와줍니다.

### JPA와 Hibernate의 관계

- JPA는 표준 인터페이스이기 때문에, 실제로 데이터베이스와 상호작용을 하려면 JPA의 구현체가 필요합니다. Hibernate는 JPA의 대표적인 구현체 중 하나로, 가장 널리 사용됩니다.

- JPA가 ORM 표준을 정의하고 있다면, Hibernate는 이 표준을 기반으로 JPA의 동작을 구현하는 라이브러리입니다. 즉, JPA는 추상적인 규칙을 제공하고, Hibernate는 이를 실제로 동작하게 만듭니다.

- 그 외에도 EclipseLink, OpenJPA 등 다른 JPA 구현체도 있지만, Spring Data JPA 환경에서는 일반적으로 Hibernate를 사용합니다.

### JPA의 주요 어노테이션

JPA에서는 객체와 데이터베이스 테이블을 매핑하기 위해 여러 가지 어노테이션을 제공합니다. 이 어노테이션들을 통해 엔티티 클래스가 테이블과 어떻게 연결되는지를 정의할 수 있습니다.

- @Entity: 이 어노테이션이 붙은 클래스는 JPA가 관리하는 엔티티임을 나타냅니다. 즉, 해당 클래스는 데이터베이스 테이블과 매핑된다는 의미입니다.

- @Table: 엔티티가 매핑될 데이터베이스 테이블의 이름을 지정합니다. 지정하지 않으면 기본적으로 엔티티 클래스 이름이 테이블 이름으로 사용됩니다.

    ```kotlin
    @Entity
    @Table(name = "users")
    class User {
        // 엔티티 클래스의 필드 정의
    }
    ```

- @Id: 해당 필드가 테이블의 기본 키(primary key)임을 나타냅니다.

- @GeneratedValue: 기본 키가 자동으로 생성되는 방식을 정의합니다. 주로 `AUTO`, `IDENTITY`, `SEQUENCE`, `TABLE` 전략이 사용됩니다.

- @Column: 엔티티의 필드를 데이터베이스의 컬럼과 매핑합니다. 컬럼 이름, 길이, null 허용 여부 등을 설정할 수 있습니다. 지정하지 않으면 필드 이름이 컬럼 이름으로 매핑됩니다.

    ```kotlin
    @Entity
    class User {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        var id: Long? = null
    
        @Column(name = "username", nullable = false)
        var username: String = ""
    }
    ```

### 엔티티(Entity)와 테이블(Table) 매핑

- JPA에서 엔티티(Entity)는 데이터베이스의 테이블(Table)과 매핑되는 자바 객체입니다. 즉, 엔티티는 데이터베이스에서 하나의 레코드(Row)를 표현하고, 엔티티의 각 필드는 테이블의 컬럼(Column)과 매핑됩니다.

- 예를 들어, `User`라는 엔티티는 데이터베이스의 `users` 테이블과 매핑되고, 엔티티의 `username` 필드는 `users` 테이블의 `username` 컬럼과 연결됩니다. 이를 통해 자바 객체를 사용하여 데이터베이스의 데이터를 쉽게 다룰 수 있게 됩니다.

- 또한, 엔티티 간의 관계도 JPA를 통해 정의할 수 있습니다. 예를 들어, `@OneToMany`, `@ManyToOne`, `@ManyToMany` 같은 애노테이션을 사용하여 테이블 간의 외래 키 관계를 객체 지향적으로 표현할 수 있습니다.

    ```kotlin
    @Entity
    @Table(name = "users")
    class User {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        var id: Long? = null
    
        @Column(name = "username", nullable = false)
        var username: String = ""
    }
    ```

이러한 매핑을 통해 JPA는 자바 애플리케이션의 객체 모델과 데이터베이스의 테이블 간의 일관된 관계를 유지하게 해줍니다. ORM의 핵심은 이런 매핑을 통해 개발자가 SQL에 의존하지 않고 객체 지향적인 방식으로 데이터베이스와 상호작용할 수 있게 하는 데 있습니다.

## **Spring Data JPA**

### Spring과 JPA의 통합

- Spring은 강력한 의존성 주입(DI)과 트랜잭션 관리 기능을 제공하는 프레임워크로, 이를 통해 JPA와의 통합이 용이합니다.

- Spring Data JPA는 Spring과 JPA를 통합하여 ORM 기능을 더 간단하고 쉽게 사용할 수 있게 해줍니다. 이를 통해 개발자는 JPA의 복잡한 설정 없이도 간단한 설정만으로 데이터베이스 연동을 할 수 있습니다.

- Spring Data JPA는 JPA의 표준을 기반으로 하며, JPA 엔티티 매핑을 활용하면서도, 다양한 리포지토리(repository) 계층의 자동화된 구현을 지원합니다. 즉, 개발자는 직접 SQL 쿼리를 작성할 필요 없이, 메서드 이름만으로도 데이터베이스 작업을 쉽게 처리할 수 있습니다.

### Spring Data JPA의 구조 및 역할

- Spring Data JPA는 데이터베이스와의 상호작용을 쉽게 처리할 수 있도록 다양한 인터페이스와 클래스들을 제공합니다.

- 주요 계층 구조

    - Repository: Repository는 가장 상위 레벨의 인터페이스입니다. Spring Data JPA의 기본적인 인터페이스로, 이 자체로는 아무 기능도 제공하지 않습니다. 그저 기본적인 리포지토리 패턴을 정의한 추상적인 인터페이스일 뿐입니다.

        이 인터페이스는 메서드가 정의되어 있지 않으며, 데이터 액세스 계층을 추상화하는 기반 역할을 합니다.

        ```kotlin
        public interface Repository<T, ID>
        ```

    - CrudRepository: 리포지토리의 기본 인터페이스로, CRUD 작업을 위한 메서드들을 제공합니다.

    - JpaRepository: CrudRepository를 확장한 인터페이스로, JPA에 특화된 추가 기능(페이징, 정렬 등)을 제공합니다.

### JpaRepository, CrudRepository 인터페이스 활용

- Spring Data JPA는 JpaRepository와 CrudRepository 인터페이스를 제공하여 개발자가 직접 구현하지 않고도 기본적인 데이터 액세스 기능을 사용할 수 있게 합니다.

- CrudRepository는 기본적인 CRUD(Create, Read, Update, Delete) 작업을 위한 메서드를 제공하는 가장 기본적인 인터페이스입니다.

    이 인터페이스는 데이터를 저장, 조회, 수정, 삭제하는 작업을 처리하기 위한 메서드들이 제공됩니다. 즉, 실제 CRUD 작업을 수행할 수 있는 기능을 가지고 있습니다.

    예를 들어, `save()`, `findById()`, `findAll()`, `deleteById()` 등의 메서드들이 정의되어 있어, 이 인터페이스를 상속받으면 별도로 구현 없이 기본적인 CRUD 작업을 사용할 수 있습니다.

    ```kotlin
    public interface CrudRepository<T, ID> extends Repository<T, ID> {
        <S extends T> S save(S entity);
        Optional<T> findById(ID id);
        Iterable<T> findAll();
        void deleteById(ID id);
        void delete(T entity);
    }
    ```

- JpaRepository는 CrudRepository를 확장하여 더 많은 기능을 제공합니다. 예를 들어, 페이징 처리, 정렬 등의 기능을 추가로 사용할 수 있습니다.

    ```kotlin
    interface JpaRepository<T, ID> : CrudRepository<T, ID> {
        fun findAll(pageable: Pageable): Page<T>
        fun findAll(sort: Sort): List<T>
    }
    ```

### 기본적인 CRUD 작업 처리

Spring Data JPA를 통해 직접 SQL을 작성하지 않고도 CRUD 작업을 쉽게 처리할 수 있습니다. `JpaRepository`나 `CrudRepository` 인터페이스를 상속받은 리포지토리를 사용하여 아래와 같은 작업을 자동으로 처리할 수 있습니다.

- Create (저장): `save()` 메서드를 사용해 엔티티를 저장합니다. 새 엔티티라면 INSERT 쿼리를 생성하고, 기존 엔티티라면 UPDATE 쿼리를 실행합니다.

    ```kotlin
    val user = User(username = "John")
    userRepository.save(user)
    ```

- Read (조회): `findById()`를 사용하여 특정 엔티티를 조회하고, `findAll()`을 사용하여 모든 엔티티를 조회할 수 있습니다. 또한, `findBy...` 형태의 메서드를 작성하여 다양한 조건으로 데이터를 조회할 수 있습니다.

    ```kotlin
    val user: Optional<User> = userRepository.findById(1L)
    val allUsers: List<User> = userRepository.findAll()
    ```

- Update (수정): `save()` 메서드는 기본적으로 엔티티가 존재하면 수정 작업을 처리합니다. 특정 엔티티를 조회한 후 필드를 변경하고 다시 저장하면 UPDATE 쿼리가 실행됩니다.

    ```kotlin
    val user = userRepository.findById(1L).orElseThrow()
    user.username = "Jane"
    userRepository.save(user)
    ```

- Delete (삭제): `deleteById()`를 사용하여 특정 엔티티를 삭제하거나, `delete()` 메서드를 사용해 엔티티를 직접 삭제할 수 있습니다.

    ```kotlin
    userRepository.deleteById(1L)
    ```



