# 12.스프링 데이터 JPA

CRUD작업이 반복됨, 이런 문제는 제너릭과 상속을 사용하여 공통화 부분을 부모 클래스(GenericDAO)로 구현하면 되지만 종속과 상속의 단점에 노출된다.

## 12.1 스프링 데이터 JPA 소개

인터페이스만 가지고도 개발을 할 수 있도록 스프링에서 제공해 준다.

![image](https://github.com/hanbroz/jpa/blob/master/12/images/img12_1.png)<br>
<스프링 데이터 JPA 사용>

스프링 데이터 JPA는 다양한 메서드를 이름 기반으로 해석하여 실행한다.

### 12.1.1 스프링 데이터 프로젝트

![image](https://github.com/hanbroz/jpa/blob/master/12/images/img12_2.png)<br>
<스프링 데이터>

# 12.2 스프링 데이터 JPA 설정

**필요한 라이브러리**

>spring-data-jpa

~~~xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

~~~

**환경설정**

~~~xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

	<jpa:repositories base-package="com.hanbroz.study.jpa.repository" />

</beans>

~~~

~~~java

@Confuguration
@EnableJpaRepository(basePackages = "com.hanbroz.study.jpa.repository")
public class AppConfig {
}

~~~

![image](https://github.com/hanbroz/jpa/blob/master/12/images/img12_3.png)<br>
<구현클래스 생성>

## 12.3 공통 인터페이스 기능

스프링 JPA를 사용하는 가장 간단한 방법은 JapReposiry 인터페이스를 상속받는 것이다.

~~~java

public interface JapRepository<T,ID extends Serializable> extends PagingAndSortingRepository<T,D> {
    ...
}

public interface MemberRepository extends JapRepository<Member, Long> {
    ...
}

~~~

이제 MemberRepository는 JpaRepository가 제공하는 다양한 기능을 사용할 수 있다.

## 12.4 쿼리 메서드 기능

쿼리메서드의 3가지 기능

* 메소드 이름으로 쿼리 생성
* 메소드 이름으로 JPA NamedQuery 호출
* @Query 어노테이션을 사용해서 Repository 인터페이스에 쿼리 직접 정의

### 12.4.1 메소드 이름으로 쿼리 생성

이메일과 이름으로 회원을 조회 하는 방법

~~~java

public interface MemberRepository extends Repository<Member, Long> {
    List<Member> findByEmailAndName(String email, String name);
}

~~~

최종 실행되는 SQL

~~~sql

select m from Member m  where m.email = ?1 and m.name = ?2;

~~~

표참고 > 12.1 스프링 데이터 JPA 쿼리 생성 기능

### 12.4.2 JPA NamedQuery

~~~java

@Entity
@NamedQuery(
        name="Member.findByUserName",
        query="select m from Member m where m.userName = :userName"
)
public class Member {
    ...
}

~~~

~~~xml

<named-query name="Member.findByUserName">
    <query><CDATA[
    select m from Member m where m.userName = :userName
    ]></query>
</named-query>

~~~

~~~java

public class MemberRepository {
    
    public List<Member> findByUserName(String userName) {
        ...
        List<Member> members =
            em.createNamedQuery("Member.findByUserName", Member.class)
            .setParameter("userName", "회원1")
            .getResultList();
    }
    
}

~~~

~~~java

public interface MemberRepository extends JapRepository<Member, Long> {
    
    List<Member> findByUserName(@Param("userName") String userName)
    
}

~~~

도메인 클래스 + . + 메서드 이름으로 Named 쿼리를 찾아서 실행한다.


### 12.4.3 @Query, Repository에 쿼리 정의

실행시점에 오류를 발견할 수 있다.

~~~java

public interface MemberRepository extends  JapRepository<Member, Long> {
    
    @Query("select m from Member m where m.userName = ?1")
    Member findbyUserName(String userName);
    
}

~~~

네이티브 SQL을 사용하려면 @Query 어노테이션에 nativeQuery = true 설정, 위치 기반은 1부터 시작하고 네이티브 SQL 예제는 0부터 시작한다.

~~~java

public interface MemberRepository extends JapRepository<Member, Long> {
    
    @Query(value="SELECT * FROM MEMBER WHERE USERNAME = ?0", nativeQuery=true)
    Member findByUserName(String userName)
    
}

~~~

### 12.4.4 파라미터 바인딩

위치, 이름 기반 바인딩 모두 지원 > 이름 기반 바인딩 추천

~~~java

import org.springframework.data.repository.query.Param

public interface MemberRepository extends JpaRepository<Member, Long> {
    
    @Query("select m from Member m where m.userName = :name")
    Member findByUserName(@Param("name") String userName);
    
}

~~~

### 12.4.5 벌크성 수정 쿼리

~~~java

int bulkPriceUp(String stockAmount) {
    ...
    String qlString = 
        "update Product p set p.price = p.price * 1.1 where p.stockAmount < :stockAmount";
    
    int resultCount = em.createQuery(qlString)
        .setParameter("stockAmount", stockAmount)
        .executeUpdate();
}

~~~

~~~java

@Modifying
@Query("update Product p set p.price = p.price * 1.1 where p.stockAmount < :stockAmount")
int bulkPriceUp(@Param("stockAmount") String stockAmount);

~~~

벌크성 쿼리를 실행 후 영속성 컨텍스트를 초기화하는 경우

>@Modifying(clearAutomatically=true)

로 설정한다. (기본값은 false)

### 12.4.6 반환타입

유연한 타입 반환 지원

~~~java

List<Member> findbyName(String name); // 컬렉션, 결과가 없으면 빈 켈력션
Member findByEmail(String email); // 단건, 결과가 없으면 NULL

~~~

### 12.4.7 페이징과 정렬

* org.springframework.data.domain.Sort : 정렬기능
* org.springframework.data.domain.Pageable : 페이지 기능 (내부에 Sort 포함)

~~~java

Page<Member> findByName(String name, Pageable pageable); // Count 쿼리 사용
List<member> findByName(String name, Pageable pageable); // Count 쿼리 사용 안함

~~~

~~~java

PageRequest pageRequest = new PageRequest(0,10, new Sort(Sort.Direction.DESC, "name"));
        
Page<Member> result = memberRepository.findByNameStartWith("김", pageRequest);

// 조회된 데이터
List<Member> members = result.getContent();
// 전체 페이지 수
int totalPages = result.getTotalPages();
// 다음 페이지 존재 여부
boolean hasNextPage = result.hasNextPage();

~~~

Pageable은 인터페이스고 사용은 구현체인 PageRequest을 사용

**Page 인터페에스**

표 참고

### 12.4.8 힌트

>import org.springframework.data.jpa.repository.QueryHints

을 사용한다. 이것은 SQL 힌트가 아니라 구현체에게 제공되는 힌트

~~~java

@QueryHints(value={@QueryHint(name="org.hibernate.readOlny", value = true)}, forCounting = true)
Page<Member> findByName(String name, Pageable pageable);

~~~

### 12.4.9 Lock

쿼리 실행 할때 Lock을 걸려면 import org.springframework.data.jpa.repository.Lock을 사용한다.

~~~java

@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Member> findByName(String name);

~~~

## 12.5 명세

명세와 술어 > 참, 거짓으로 평가됨 (AND, OR와 같은 연산자로 조합할 수 있다.)

org.springframework.data.jpa.domain.Specification : 술어, 컴포지트 패턴으로 정의 되어 여러 Specification을 조합하여 다양한 검색 조건을 만들어 낼 수 있다.
사용하려면 org.springframework.data.jpa.repository.JpaSpecificationExecutor을 상속 받으면 된다.

~~~java

public interface OrderRepository extends JpaRepository<Order, Long>, JpaSpecificationExecutor<Order> {  
    ...
}

~~~

Specification는 명세를 조립할 수 있도록 도와주는 클래스 where(), and(), or(), not() 메서드를 제공한다.

import static을 적용하면 더 읽기 쉬운 코드가 된다.

~~~java

public class OrderSpec {

    public static Specification<Order> memberNameLike(final String memberName) {
        return new Specification<Order>() {
            public Predicate toPredicate(Root<Order> root, CriteriaQuery<?> query, CriteriaBuilder builder) {

                if (StringUtils.isEmpty(memberName)) return null;

                Join<Order, Member> m = root.join("member", JoinType.INNER); //회원과 조인
                return builder.like(m.<String>get("name"), "%" + memberName + "%");
            }
        };
    }

    public static Specification<Order> orderStatusEq(final OrderStatus orderStatus) {
        return new Specification<Order>() {
            public Predicate toPredicate(Root<Order> root, CriteriaQuery<?> query, CriteriaBuilder builder) {

                if (orderStatus == null) return null;

                return builder.equal(root.get("status"), orderStatus);
            }
        };
    }
}

~~~

명세를 구성하기 위해 Specification 인터페이스를 구현했다.

## 12.6 사용자 정의 리포지토리 구현

메서드를 직접 구현해야 하는 경우가 존재, 필요한 메소드만 구현할 수 있는 방법

사용자 인터페이스를 구성한다.

~~~java

public interface MemberRepositoryCustom {
    public List<Member> findMemberCustomer();
}

~~~

구현할 클래스를 작성, 규칙이 있다. 레포지토리 인터페이스 이름 + Imple로 짓는다.

~~~java

public class MemberRepositoryImpl implements MemberRepositoryCustom {
    
    @Override
    public List<Member> findMemberCustomer() {
        // 사용자 정의 구현
    }
    
}

~~~

Impl외 다른 단어를 붙이고 싶다면 repository-impl-postfix에서 변경 가능

## 12.7 Web 확장

MVC에서 사용할 수 있는 편리한 기능 제공

### 12.7.1 설정

Web 확장 기능을 활성화 하려면

>org.springframework.data.web.config.SpringDataWebConfiguration

을 빈으로 등록

### 12.7.2 도메인 클래스 컨버터 기능

/memberUpdateForm?id=1

~~~java

@Controller
public class MemberController {

    @Autowired
    MemberRepository memberRepository;

    @RequestMapping(value = "/members/memberUpdateForm")
    public String memberUpdateForm(@RequestParam("id") Long id, Model model) {
        
        Member member = memberRepository.findOne(id);
        model.addAttribute("member", member);
        return "/memberServiceForm";
        
    }
    
    @RequestMapping(value = "/members/memberUpdateForm")
    public String memberUpdateForm(@RequestParam("id") Member member) { // 도메인 컨버터가 적용됨
        
        model.addAttribute("member", member);
        return "/memberServiceForm";
        
    }

}

~~~

### 12.7.3 페이징 정렬 기능

HandlerMethodArgumentResolver을 통해 MVC에서 페이지과 정렬 기능 제공

* 페이징 기능 : PageableHandlerMethodArgumentResolver
* 정렬 기능 : SortHandlerMethodArgumentResolver

~~~java

@RequestMapping(value = "/members",method = RequestMethod.GET)
public String list(Pageable pageable, Model model) {

    Page<Member> page = memberService.findMembers(pageable);
    model.addAttribute("member", page.getContent());
    return "members/memberList";
}

~~~

**접두사**

@Qualifier 사용, "{접두사명}_"로 구분한다.

>members?member_page=0&order_page=1

**기본값**

Pageable의 기본값은 page=0, size=20이다. 변경하고 싶으면 @PageableDefault 어노테이션을 사용한다.

## 12.8 스프링 데이터 JPA가 사용하는 구현체

