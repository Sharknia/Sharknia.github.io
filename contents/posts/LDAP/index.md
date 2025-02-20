---
IDX: "NUM-250"
tags:
  - BackEnd
description: "LDAP(Lightweight Directory Access Protocol)"
update: "2025-02-20T09:36:00.000Z"
date: "2025-02-20"
상태: "Ready"
title: "LDAP"
---
![](image1.png)
## 서론

LDAP에 대해 알아보겠습니다. 개념적인 부분들에 대해 정리하고 넘어가려고 합니다. 

## 디렉터리 서비스

LDAP은 디렉터리 서비스에 저장된 데이터를 검색하고 관리하기 위한 프로토콜입니다. LDAP을 이해하려면 먼저 디렉터리 서비스가 무엇인지 알아야 합니다.

### 디렉터리 서비스(Directory Service)란? 

디렉터리 서비스는 조직 내의 사용자, 그룹, 기기 등의 정보를 저장하고 관리하는 시스템이며, 이를 효율적으로 조회하고 인증하는 데 LDAP 같은 프로토콜이 사용됩니다. 쉽게 이야기하면 디렉터리 서비스는 회사 전화번호부로 비유할 수 있습니다. 

- 직원들의 이름, 직급, 이메일, 전화번호 등을 한 곳에 정리해 두는 역할을 합니다.

- 누군가 특정 사람의 정보를 찾고 싶다면 전화번호부에서 이름을 검색하면 해당 정보를 찾을 수 있습니다. 

- 즉, 디렉터리 서비스는 조직 내의 중요한 정보(사용자, 그룹, 권한, 컴퓨터, 네트워크 장비 등)를 저장하고 빠르게 검색할 수 있는 시스템입니다.

### 디렉터리 서비스의 특징

일반적인 DB와 비교하면 다음과 같습니다. 

| 특징 | 디렉터리 서비스 | 일반 데이터베이스 (RDB) |
| --- | --- | --- |
| **구조** | 계층적(Tree) 구조 | 테이블 기반 (관계형) |
| **목적** | 빠른 검색과 조회 | 데이터 저장, 분석 |
| **사용 예시** | 사용자 인증, 조직 관리 | 금융 데이터, 거래 내역 |
| **데이터 변경** | 변경이 적음 (읽기 위주) | 잦은 수정, 삽입, 삭제 |
| **예제** | Active Directory, OpenLDAP | MySQL, PostgreSQL |

디렉터리 서비스는 특히 자주 변경되지 않는 정보를 저장하고 빠르게 조회하는데에 최적화 되어있습니다. 

DynamoDB 같은 일반 데이터베이스를 써도 사용자 정보를 저장하고 빠르게 조회할 수 있습니다. 그렇다면 왜 굳이 LDAP 같은 디렉터리 서비스가 필요할까요? 디렉터리 서비스가 가진 특화된 기능 때문입니다.

### 디렉터리 서비스의 강점

디렉터리 서비스는 단순한 “빠른 조회”를 넘어서, 계층 구조, 인증, 권한 관리, 표준화된 연동이 가능하도록 특화되어 있어 다음과 같은 강점을 가집니다. 물론 DynamoDB, PostgreSQL등 일반 DB로도 구현이 모두 가능하지만 불리한 점이 많습니다. 

기업 환경에서 “조직 관리 + 인증 + 보안”까지 고려하면 LDAP 기반 디렉터리 서비스가 훨씬 유리합니다.

#### **계층적 구조**

조직의 구조(부서, 그룹, 직급 등)을 반영할 수 있습니다. (예: cn=사용자1, ou=개발팀, dc=company, dc=com)

```plain text
dc=company, dc=com
 ├── ou=IT부서
 │    ├── cn=김개발 (개발자)
 │    ├── cn=이엔지 (엔지니어)
 │
 ├── ou=인사팀
 │    ├── cn=박HR (인사 담당)
 │    ├── cn=최총무 (총무 담당)
 │
 ├── ou=경영진
      ├── cn=정대표 (CEO)
      ├── cn=한이사 (이사)
```

이렇게 계층적인 구조를 통해 조직을 명확하게 구분하고, 특정 그룹이나 사용자에 대한 접근 권한을 관리할 수 있는 것이 디렉터리 서비스의 강점입니다.

#### 빠른 읽기 조회

데이터가 자주 변경되지 않는 것을 전제하므로 읽기 성능이 최적화되어있습니다.

## LDAP(Lightweight Directory Access Protocol)

디렉터리 서비스에 대해 알게 됐으니 이제 이에 접근하는 프로토콜인 LDAP 이 뭔지 정리해보겠습니다. 

### LDAP이란? 

LDAP은 디렉터리 서비스와 클라이언트가 통신하는 방법을 정의합니다.

LDAP은 경량화된 프로토콜로, TCP/IP를 기반으로 동작하며, 네트워크 상에서 디렉터리 정보를 효율적으로 조회할 수 있도록 설계되었습니다. 기업 환경에서는 사용자 인증, 권한 관리, 싱글사인온(SSO) 등의 목적으로 활용됩니다.

### LDAP의 특징

#### **경량 프로토콜**

네트워크에서 최소한의 리소스로 동작하도록 설계되었습니다.

#### 표준화된 프로토콜

LDAP을 지원하는 다양한 시스템에서 동일한 방식으로 연동할 수 있습니다.

#### 인증 및 인가 기능의 내장

사용자 인증/인가에 관련된 기능이 내장되어 있습니다. 

#### SSO 지원

LDAP 기반으로 여러 시스템을 한 번의 로그인으로 사용할 수 있습니다. 

### LDAP의 구조

#### **디렉터리 정보 트리(DIT, Directory Information Tree)**

LDAP의 데이터는 디렉터리 정보 트리(DIT) 형태로 구성됩니다. 이는 디렉터리 서비스 내 데이터를 계층적(Tree)으로 정리하는 방식으로, 조직의 사용자, 그룹, 기기 등의 관계를 표현할 수 있습니다.

다음은 조직 중심으로 설계된 예제 구조 트리입니다. 

```plain text
dc=company, dc=com
 ├── ou=IT부서
 │    ├── cn=김개발 (개발자)
 │    ├── cn=이엔지 (엔지니어)
 │
 ├── ou=인사팀
 │    ├── cn=박HR (인사 담당)
 │    ├── cn=최총무 (총무 담당)
 │
 ├── ou=경영진
      ├── cn=정대표 (CEO)
      ├── cn=한이사 (이사)
```

LDAP을 사용하는 기업이나 조직마다 사용자, 그룹, 기기 등을 관리하는 방식이 다르기 때문에 디렉터리 구조도 달라질 수 있습니다. 다음은 지역 중심으로 설계된 예제 구조 트리입니다. 

```plain text
dc=companyB, dc=com
 ├── ou=서울
 │    ├── cn=박서울직원
 │
 ├── ou=부산
      ├── cn=최부산직원
```

즉, LDAP을 사용한다고 해서 모든 디렉터리 트리가 똑같지는 않으며, 조직의 정책에 따라 다르게 설계될 수 있습니다.

또 LDAP을 사용하는 대표적인 디렉터리 서비스들(Active Directory, OpenLDAP 등)은 기본적으로 제공하는 트리 구조(기본 DIT)가 다르기 때문에 트리 구조가 달라질 수도 있습니다. 

따라서 기본적으로 LDAP 트리는 조직의 관리 정책을 반영하여 설계자가 LDAP 스키마(objectClass, attribute 등)를 기반으로 직접 정의해야 합니다. 

### **LDAP 스키마(objectClass & attribute)**

LDAP 스키마는 어떤 종류의 데이터를 저장할 것인지 미리 정의한 규칙입니다. 스키마는 objectClass와 attribute로 구성됩니다.

#### **objectClass (객체 클래스)**

LDAP 엔트리가 어떤 유형인지 정의하는 역할을 합니다. 하나의 엔트리는 하나 이상의 objectClass를 가져야 하고, objectClass는 필수 속성과 선택 속성을 지정할 수 있습니다. 

```plain text
objectClass ( 2.16.840.1.113730.3.2.2
    NAME 'inetOrgPerson'
    DESC 'User entry with common attributes'
    SUP organizationalPerson
    STRUCTURAL
    MUST ( sn $ cn )
    MAY ( mail $ telephoneNumber )
)
```

sn, cn은 필수 속성으로 반드시 포함해야 하며 mail, telephoneNumber은 선택 속성으로 필수가 아닙니다. 

#### attribute (속성)

각 엔트리가 가질 수 있는 데이터 값(필드)입니다. 예를 들어 위에서 cn, sn, mail, uid, telephoneNumber 등입니다. 

```plain text
attributeType ( 2.5.4.3
    NAME 'cn'
    DESC 'Common Name'
    EQUALITY caseIgnoreMatch
    SUBSTR caseIgnoreSubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{64}
)
```

- NAME ‘cn’ → 속성 이름

- SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 → 문자열 타입 속성

- EQUALITY caseIgnoreMatch → 대소문자 무시하고 비교

### LDAP 엔트리

LDAP 엔트리는 하나의 개체(사용자, 그룹, 장비 등)를 의미하는 데이터 단위입니다. 각 엔트리는 objectClass에 따라 정해진 attribute를 포함해야 합니다.

```plain text
dn: cn=홍길동, ou=사용자, dc=company, dc=com
objectClass: inetOrgPerson
cn: 홍길동
sn: 길동
mail: hong@company.com
telephoneNumber: 010-1234-5678
```

### LDAP의 주요 동작

#### **바인딩 (Bind)**

LDAP 서버에 연결 및 인증하는 과정입니다. 

- 익명 바인딩: 인증 없이 연결

- 단순 바인딩: 사용자 DN과 비밀번호를 사용한 인증

- SASL 바인딩: 보안 강화를 위한 인증방식

#### 검색 (Search)

LDAP에서 특정 데이터를 검색하는 과정입니다. 예를 들어, 특정 사용자를 찾을 때 다음과 같은 요청을 보냅니다.

```plain text
ldapsearch -x -b "dc=company,dc=com" "(cn=김개발)"
```

- `(cn=김개발)`: 사용자 이름이 "김개발"인 엔트리 검색

- `(objectClass=person)`: 모든 사용자 정보 검색

#### 추가 (Add)

새로운 엔트리를 디렉터리에 추가하는 과정입니다.

```plain text
dn: cn=새사용자, ou=IT부서, dc=company, dc=com
objectClass: inetOrgPerson
cn: 새사용자
sn: 김철수
mail: newuser@company.com
```

#### 수정 (Modify)

기존 엔트리의 속성을 변경하는 과정입니다.

```plain text
ldapmodify -x -D "cn=admin,dc=company,dc=com" -W <<EOF
modify dn: cn=김개발, ou=IT부서, dc=company, dc=com
replace: mail
mail: newemail@company.com
EOF
```

#### 삭제 (Delete)

엔트리를 삭제하는 과정입니다.

```plain text
ldapdelete -x "cn=김개발,ou=IT부서,dc=company,dc=com"
```

### LDAP의 활용

LDAP은 주로 다음과 같은 역할을 합니다. 

#### 사용자 인증 **(Authentication)**

직원이 회사 시스템에 로그인할 때, 디렉터리 서비스에서 ID/PW를 확인하고 인증을 수행할 수 있습니다. 

#### **권한 관리 (Authorization)**

특정 직원이 특정 시스템에 접근할 수 있는지 확인합니다. 

#### **검색 및 조회**

직원의 이메일, 부서, 역할 등을 빠르게 찾을 수 있습니다. 

## LDAP과 디렉터리 서비스

처음에 잘 이해가 되지 않았지만, 이렇게 생각하니 좀 이해가 쉬워진 것 같습니다. 

디렉터리 서비스는 읽기와 검색에 최적화 되어있는 데이터베이스(저장소)의 일종이고, LDAP은 그러한 디렉터리 서비스의 스키마를 정의하고, 계층적 구조와 접근 방식(쿼리 언어 등)을 규정하는 역할을 합니다. 관계형 데이터베이스에서 스키마가 테이블 구조와 SQL 쿼리를 정형화하는 것과 유사하게 LDAP 스키마는 디렉터리 서비스의 데이터 구조와 검색 방법을 정형화해 줍니다.

즉 LDAP이라는 표준화된 접근방식인 프로토콜이 정의되어 있지 않았다면 디렉터리 서비스는 단순 저장소에 불과하므로 다양한 시스템에서 동일한 방식으로 연동하거나 하는 일이 불가능할 것입니다. 



