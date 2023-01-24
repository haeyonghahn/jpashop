# jpashop
## 목차
* **[스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발](#스프링-부트와-JPA-활용1---웹-애플리케이션-개발)**
* **[스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](#스프링-부트와-JPA-활용2---API-개발과-성능-최적화)**
  * **[API 개발 기본](#API-개발-기본)**
    * **[회원 등록 API](#회원-등록-API)**
    * **[회원 수정 API](#회원-수정-API)**
    * **[회원 조회 API](#회원-조회-API)**
  * **[API 개발 고급 - 준비](#API-개발-고급---준비)**
    * **[조회용 샘플 데이터 입력](#조회용-샘플-데이터-입력)**
  * **[API 개발 고급 - 지연 로딩과 조회 성능 최적화](#API-개발-고급---지연-로딩과-조회-성능-최적화)**
    * **[간단한 주문 조회 V1: 엔티티를 직접 노출](#간단한-주문-조회-v1-엔티티를-직접-노출)**
    * **[간단한 주문 조회 V2: 엔티티를 DTO로 변환](#간단한-주문-조회-V2-엔티티를-DTO로-변환)**
    * **[간단한 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화](#간단한-주문-조회-V3-엔티티를-DTO로-변환---페치-조인-최적화)**
    * **[간단한 주문 조회 V4: JPA에서 DTO로 바로 조회](#간단한-주문-조회-V4-JPA에서-DTO로-바로-조회)**
  * **[API 개발 고급 - 컬렉션 조회 최적화](#API-개발-고급---컬렉션-조회-최적화)**
    * **[주문 조회 V1: 엔티티 직접 노출](#주문-조회-V1-엔티티-직접-노출)**
    * **[주문 조회 V2: 엔티티를 DTO로 변환](#주문-조회-V2-엔티티를-DTO로-변환)**
    * **[주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화](#주문-조회-V3-엔티티를-DTO로-변환---페치-조인-최적화)**
    * **[주문 조회 V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파](#주문-조회-V31-엔티티를-DTO로-변환---페이징과-한계-돌파)**

## 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발
## 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화
### API 개발 기본
#### 회원 등록 API
__V1 엔티티를 Request Body에 직접 매핑__   
```java
package jpabook.jpashop.api;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.service.MemberService;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequiredArgsConstructor
public class MemberApiController {

    private final MemberService memberService;

    /**
     * 등록 V1: 요청 값으로 Member 엔티티를 직접 받는다.
     * 문제점
     * - 엔티티에 프레젠테이션 계층을 위한 로직이 추가된다.
     *   - 엔티티에 API 검증을 위한 로직이 들어간다. (@NotEmpty 등등)
     *   - 실무에서는 회원 엔티티를 위한 API가 다양하게 만들어지는데, 
     *     한 엔티티에 각각의 API를 위한 모든 요청 요구사항을 담기는 어렵다.
     * - 엔티티가 변경되면 API 스펙이 변한다.
     * 결론
     * - API 요청 스펙에 맞추어 별도의 DTO를 파라미터로 받는다.
     */
    @PostMapping("/api/v1/members")
    public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @Data
    static class CreateMemberRequest {
        private String name;
    }

    @Data
    static class CreateMemberResponse {
        private Long id;

        public CreateMemberResponse(Long id) {
            this.id = id;
        }
    }
}
```
- 문제점
  - 엔티티에 프레젠테이션 계층을 위한 로직이 추가된다.
  - 엔티티에 API 검증을 위한 로직이 들어간다. (@NotEmpty 등등)
  - 실무에서는 회원 엔티티를 위한 API가 다양하게 만들어지는데, 한 엔티티에 각각의 API를 위한
  모든 요청 요구사항을 담기는 어렵다.
  - 엔티티가 변경되면 API 스펙이 변한다.
- 결론
  - API 요청 스펙에 맞추어 별도의 DTO를 파라미터로 받는다.

__V2 엔티티 대신에 DTO를 RequestBody에 매핑__
```java
/**
 * 등록 V2: 요청 값으로 Member 엔티티 대신에 별도의 DTO를 받는다.
 */
@PostMapping("/api/v2/members")
public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {

    Member member = new Member();
    member.setName(request.getName());

    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}
```
- `CreateMemberRequest` 를 `Member` 엔티티 대신에 `RequestBody` 와 매핑한다.
- 엔티티와 프레젠테이션 계층을 위한 로직을 분리할 수 있다.
- 엔티티와 API 스펙을 명확하게 분리할 수 있다.
- 엔티티가 변해도 API 스펙이 변하지 않는다.
> 참고 : 실무에서는 엔티티를 API 스펙에 노출하면 안된다!

#### 회원 수정 API
```java
/**
 * 수정 API
 */
@PutMapping("/api/v2/members/{id}")
public UpdateMemberResponse updateMemberV2(@PathVariable("id") Long id, @RequestBody @Valid UpdateMemberRequest request) {
    memberService.update(id, request.getName());
    Member findMember = memberService.findOne(id);
    return new UpdateMemberResponse(findMember.getId(), findMember.getName());
}


@Data
static class UpdateMemberRequest {
    private String name;
}

@Data
@AllArgsConstructor
static class UpdateMemberResponse {
    private Long id;
    private String name;
}
```
- 회원 수정도 DTO를 요청 파라미터에 매핑

```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.repository.MemberRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    ...
    
    /**
     * 회원 수정
     */
    @Transactional
    public void update(Long id, String name) {
        Member member = memberRepository.findOne(id);
        member.setName(name);
    }

}
```
- 변경 감지를 사용해서 데이터를 수정
> 오류정정: 회원 수정 API updateMemberV2 은 회원 정보를 부분 업데이트 한다. 여기서 PUT 방식을
사용했는데, PUT은 전체 업데이트를 할 때 사용하는 것이 맞다. 부분 업데이트를 하려면 PATCH를
사용하거나 POST를 사용하는 것이 REST 스타일에 맞다.

#### 회원 조회 API
> 중요 : `request`, `response` 에 절대. 아니 강제이다. 엔티티를 노출하지말고 Dto 를 이용해라!

__회원조회 V1: 응답 값으로 엔티티를 직접 외부에 노출__
```java
package jpabook.jpashop.api;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.service.MemberService;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequiredArgsConstructor
public class MemberApiController {
    /**
     * 조회 V1: 응답 값으로 엔티티를 직접 외부에 노출한다.
     * 문제점
     * - 엔티티에 프레젠테이션 계층을 위한 로직이 추가된다.
     *   - 기본적으로 엔티티의 모든 값이 노출된다.
     *   - 응답 스펙을 맞추기 위해 로직이 추가된다. (@JsonIgnore, 별도의 뷰 로직 등등)
     *   - 실무에서는 같은 엔티티에 대해 API가 용도에 따라 다양하게 만들어지는데, 한 엔티티에 각각의 API를 위한 프레젠테이션 응답 로직을 담기는 어렵다.
     * - 엔티티가 변경되면 API 스펙이 변한다.
     * - 추가로 컬렉션을 직접 반환하면 항후 API 스펙을 변경하기 어렵다.(별도의 Result 클래스 생성으로 해결)
     * 결론
     * - API 응답 스펙에 맞추어 별도의 DTO를 반환한다.
     */
    //조회 V1: 안 좋은 버전, 모든 엔티티가 노출, @JsonIgnore -> 이건 정말 최악, api가 이거 하나인가! 화면에 종속적이지 마라!
    @GetMapping("/api/v1/members")
    public List<Member> membersV1() {
        return memberService.findMembers();
    }
    ...
}
```
- 문제점
  - 엔티티에 프레젠테이션 계층을 위한 로직이 추가된다.
  - 기본적으로 엔티티의 모든 값이 노출된다.
  - 응답 스펙을 맞추기 위해 로직이 추가된다. (@JsonIgnore, 별도의 뷰 로직 등등)
  - 실무에서는 같은 엔티티에 대해 API가 용도에 따라 다양하게 만들어지는데, 한 엔티티에 각각의
  API를 위한 프레젠테이션 응답 로직을 담기는 어렵다.
  - 엔티티가 변경되면 API 스펙이 변한다.
  - 추가로 컬렉션을 직접 반환하면 항후 API 스펙을 변경하기 어렵다.(별도의 Result 클래스 생성으로 해결)
- 결론
  - API 응답 스펙에 맞추어 별도의 DTO를 반환한다.

> 참고: 엔티티를 외부에 노출하지 마세요!    
> 실무에서는 `member` 엔티티의 데이터가 필요한 API가 계속 증가하게 된다. 어떤 API는 `name` 필드가 필요하지만, 어떤 API는 `name` 필드가 필요없을 수 있다. 
> 결론적으로 엔티티 대신에 API 스펙에 맞는 별도의 DTO를 노출해야 한다

__회원조회 V2: 응답 값으로 엔티티가 아닌 별도의 DTO 사용__
```java
package jpabook.jpashop.api;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.service.MemberService;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequiredArgsConstructor
public class MemberApiController {
    ...
    
    /**
     * 조회 V2: 응답 값으로 엔티티가 아닌 별도의 DTO를 반환한다.
     */
    @GetMapping("/api/v2/members")
    public Result membersV2() {

        List<Member> findMembers = memberService.findMembers();
        //엔티티 -> DTO 변환
        List<MemberDto> collect = findMembers.stream()
                .map(m -> new MemberDto(m.getName()))
                .collect(Collectors.toList());

        return new Result(collect.size(), collect);
    }

    @Data
    @AllArgsConstructor
    static class Result<T> {
        private int count;
        private T data;
    }

    @Data
    @AllArgsConstructor
    static class MemberDto {
        private String name;
    }
    ...
}
```
- 엔티티를 DTO로 변환해서 반환한다.
- 엔티티가 변해도 API 스펙이 변경되지 않는다.
- 추가로 `Result` 클래스로 컬렉션을 감싸서 향후 필요한 필드를 추가할 수 있다.

### API 개발 고급 - 준비
#### 조회용 샘플 데이터 입력
API 개발 고급 설명을 위해 샘플 데이터를 입력하자.
- userA
  - JPA1 BOOK
  - JPA2 BOOK
- userB
  - SPRING1 BOOK
  - SPRING2 BOOK

```java
package jpabook.jpashop;

import jpabook.jpashop.domain.*;
import jpabook.jpashop.domain.item.Book;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.PostConstruct;
import javax.persistence.EntityManager;

/**
 * 종 주문 2개
 * * userA
 * 	 * JPA1 BOOK
 * 	 * JPA2 BOOK
 * * userB
 * 	 * SPRING1 BOOK
 * 	 * SPRING2 BOOK
 */
@Component
@RequiredArgsConstructor
public class InitDb {

    private final InitService initService;

    @PostConstruct
    public void init() {
        initService.dbInit1();
        initService.dbInit2();
    }

    @Component
    @Transactional
    @RequiredArgsConstructor
    static class InitService {

        private final EntityManager em;

        public void dbInit1() {
            System.out.println("Init1" + this.getClass());
            Member member = createMember("userA", "서울", "1", "1111");
            em.persist(member);

            Book book1 = createBook("JPA1 BOOK", 10000, 100);
            em.persist(book1);

            Book book2 = createBook("JPA2 BOOK", 20000, 100);
            em.persist(book2);

            OrderItem orderItem1 = OrderItem.createOrderItem(book1, 10000, 1);
            OrderItem orderItem2 = OrderItem.createOrderItem(book2, 20000, 2);

            Delivery delivery = createDelivery(member);
            Order order = Order.createOrder(member, delivery, orderItem1, orderItem2);
            em.persist(order);
        }

        public void dbInit2() {
            Member member = createMember("userB", "진주", "2", "2222");
            em.persist(member);

            Book book1 = createBook("SPRING1 BOOK", 20000, 200);
            em.persist(book1);

            Book book2 = createBook("SPRING2 BOOK", 40000, 300);
            em.persist(book2);

            OrderItem orderItem1 = OrderItem.createOrderItem(book1, 20000, 3);
            OrderItem orderItem2 = OrderItem.createOrderItem(book2, 40000, 4);

            Delivery delivery = createDelivery(member);
            Order order = Order.createOrder(member, delivery, orderItem1, orderItem2);
            em.persist(order);
        }

        private Member createMember(String name, String city, String street, String zipcode) {
            Member member = new Member();
            member.setName(name);
            member.setAddress(new Address(city, street, zipcode));
            return member;
        }

        private Book createBook(String name, int price, int stockQuantity) {
            Book book1 = new Book();
            book1.setName(name);
            book1.setPrice(price);
            book1.setStockQuantity(stockQuantity);
            return book1;
        }

        private Delivery createDelivery(Member member) {
            Delivery delivery = new Delivery();
            delivery.setAddress(member.getAddress());
            return delivery;
        }
    }
}
```
> 참고: 주문 내역 화면에서는 회원당 주문 내역을 하나만 출력했으므로 하나만 노출된다.

### API 개발 고급 - 지연 로딩과 조회 성능 최적화
주문 + 배송정보 + 회원을 조회하는 API를 만들자.   
지연 로딩 때문에 발생하는 성능 문제를 단계적으로 해결해보자.   

> 참고: 지금부터 설명하는 내용은 정말 중요합니다. 실무에서 JPA를 사용하려면 100% 이해해야 한다.
> 안그러면 엄청난 시간을 날리고 강사를 원망하면서 인생을 허비하게 된다.

#### 간단한 주문 조회 V1: 엔티티를 직접 노출
__OrderSimpleApiController__
```java
package jpabook.jpashop.api;

import jpabook.jpashop.domain.Order;
import jpabook.jpashop.repository.OrderRepository;
import jpabook.jpashop.repository.OrderSearch;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

/**
 *
 * xToOne(ManyToOne, OneToOne) 관계 최적화
 * Order
 * Order -> Member
 * Order -> Delivery
 *
 */
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {

    private final OrderRepository orderRepository;

    /**
     * V1. 엔티티 직접 노출
     * - Hibernate5Module 모듈 등록, LAZY=null 처리
     * - 양방향 관계 문제 발생 -> @JsonIgnore로 해결
     */
    @GetMapping("/api/v1/simple-orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAllByString(new OrderSearch());
        for(Order order : all) {
            order.getMember().getName();        // Lazy 강제 초기화
            order.getDelivery().getAddress();   // Lazy 강제 초기화
        }
        return all;
    }
}
```
- 엔티티를 직접 노출하는 것은 좋지 않다. (앞장에서 이미 설명)
- `order` => `member` 와 `order` => `address` 는 지연 로딩이다. 따라서 실제 엔티티 대신에 프록시 존재
- jackson 라이브러리는 기본적으로 이 프록시 객체를 json으로 어떻게 생성해야 하는지 모름 예외 발생
- `Hibernate5Module` 을 스프링 빈으로 등록하면 해결(스프링 부트 사용중)

__하이버네이트 모듈 등록__    
스프링 부트 버전에 따라서 모듈 등록 방법이 다르다. 스프링 부트 3.0 부터는 `javax -> jakarta` 로 변경되어서 지원 모듈도 다른 모듈을 등록해야 한다.

__스프링 부트 3.0 미만: Hibernate5Module 등록__    
`build.gradle` 에 다음 라이브러리를 추가하자.   
`implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'`

`JpashopApplication` 에 다음 코드를 추가하자.
```java
@Bean
Hibernate5Module hibernate5Module() {
 return new Hibernate5Module();
}
```
- 기본적으로 초기화 된 프록시 객체만 노출, 초기화 되지 않은 프록시 객체는 노출 안함

만약 스프링 부트 3.0 이상을 사용하면 다음을 참고해서 모듈을 변경해야 한다. 그렇지 않으면 다음과 같은 예외가 발생한다.
`java.lang.ClassNotFoundException: javax.persistence.Transient`

__스프링 부트 3.0 이상: Hibernate5JakartaModule 등록__    
`build.gradle` 에 다음 라이브러리를 추가하자.   
`implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5-jakarta'`

`JpashopApplication` 에 다음 코드를 추가하자.
```java
@Bean
Hibernate5JakartaModule hibernate5Module() {
 return new Hibernate5JakartaModule();
}
```
- 기본적으로 초기화 된 프록시 객체만 노출, 초기화 되지 않은 프록시 객체는 노출 안함

__다음과 같이 설정하면 강제로 지연 로딩 가능__    
```java
@Bean
Hibernate5Module hibernate5Module() {
 Hibernate5Module hibernate5Module = new Hibernate5Module();
 //강제 지연 로딩 설정
 hibernate5Module.configure(Hibernate5Module.Feature.FORCE_LAZY_LOADING, true);
 return hibernate5Module;
}
```
- 옵션을 키면 `order` -> `member`, `member` -> `order` 양방향 연관관계를 계속 로딩하게 된다. 따라서 `@JsonIgnore` 옵션을 한곳에 주어야 한다.

> 주의: 스프링 부트 3.0 이상이면 `Hibernate5Module` 대신에 `Hibernate5JakartaModule` 을 사용해야 한다.

> 주의: 엔티티를 직접 노출할 때는 양방향 연관관계가 걸린 곳은 꼭! 한곳을 `@JsonIgnore` 처리 해야 한다. 안그러면 양쪽을 서로 호출하면서 무한 루프가 걸린다. 

> 참고: 앞에서 계속 강조했듯이 정말 간단한 애플리케이션이 아니면 엔티티를 API 응답으로 외부로 노출하는 것은 좋지 않다. 따라서 `Hibernate5Module` 를 사용하기 보다는 DTO로 변환해서 반환하는 것이 더 좋은 방법이다.

> 주의: 지연 로딩(LAZY)을 피하기 위해 즉시 로딩(EARGR)으로 설정하면 안된다! 즉시 로딩 때문에 연관관계가 필요 없는 경우에도 데이터를 항상 조회해서 성능 문제가 발생할 수 있다. 즉시 로딩으로 설정하면 성능 튜닝이 매우 어려워 진다. 항상 지연 로딩을 기본으로 하고, 성능 최적화가 필요한 경우에는 페치 조인(fetch join)을 사용해라!(V3에서 설명)

#### 간단한 주문 조회 V2: 엔티티를 DTO로 변환
__OrderSimpleApiController - 추가__
```java
/**
 * V2. 엔티티를 조회해서 DTO로 변환(fetch join 사용X)
 * - 단점: 지연로딩으로 쿼리 N번 호출
 */
@GetMapping("/api/v2/simple-orders")
public List<SimpleOrderDto> ordersV2() {
    List<Order> orders = orderRepository.findAll();
    List<SimpleOrderDto> result = orders.stream()
            .map(o -> new SimpleOrderDto(o))
            .collect(toList());

    return result;
}

@Data
static class SimpleOrderDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public SimpleOrderDto(Order order) {
        orderId = order.getId();
        name = order.getMember().getName();
        orderDate = order.getOrderDate();
        orderStatus = order.getStatus();
        address = order.getDelivery().getAddress();
    }
}
```
- 엔티티를 DTO로 변환하는 일반적인 방법이다.
- 쿼리가 총 1 + N + N번 실행된다. (v1과 쿼리수 결과는 같다.)
- `order` 조회 1번(order 조회 결과 수가 N이 된다.)
- `order -> member` 지연 로딩 조회 N 번
- `order -> delivery` 지연 로딩 조회 N 번
- 예) order의 결과가 4개면 최악의 경우 1 + 4 + 4번 실행된다.(최악의 경우)
  - 지연로딩은 영속성 컨텍스트에서 조회하므로, 이미 조회된 경우 쿼리를 생략한다.

#### 간단한 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화
__OrderSimpleApiController - 추가__
```java
/**
 * V3. 엔티티를 조회해서 DTO로 변환(fetch join 사용O)
 * - fetch join으로 쿼리 1번 호출
 * 참고: fetch join에 대한 자세한 내용은 JPA 기본편 참고(정말 중요함)
 */
@GetMapping("/api/v3/simple-orders")
public List<SimpleOrderDto> ordersV3() {
    List<Order> orders = orderRepository.findAllWithMemberDelivery();
    List<SimpleOrderDto> result = orders.stream()
            .map(o -> new SimpleOrderDto(o))
            .collect(toList());
    return result;
}
```
__OrderRepository - 추가 코드__
```java
public List<Order> findAllWithMemberDelivery() {
    return em.createQuery(
            "select o from Order o" +
                    " join fetch o.member m" +
                    " join fetch o.delivery d", Order.class)
            .getResultList();
}
```
- 엔티티를 페치 조인(fetch join)을 사용해서 쿼리 1번에 조회
- 페치 조인으로 `order -> member` , `order -> delivery` 는 이미 조회 된 상태 이므로 지연로딩X

#### 간단한 주문 조회 V4: JPA에서 DTO로 바로 조회
__OrderSimpleApiController - 추가__
```java
private final OrderSimpleQueryRepository orderSimpleQueryRepository; //의존관계 주입

/**
 * V4. JPA에서 DTO로 바로 조회
 * - 쿼리 1번 호출
 * - select 절에서 원하는 데이터만 선택해서 조회
 */
@GetMapping("/api/v4/simple-orders")
public List<OrderSimpleQueryDto> ordersV4() {
    return orderSimpleQueryRepository.findOrderDtos();
}
```
__OrderSimpleQueryRepository 조회 전용 리포지토리__
```java
package jpabook.jpashop.repository.order.simplequery;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;

import javax.persistence.EntityManager;
import java.util.List;

@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {

    private final EntityManager em;

    public List<OrderSimpleQueryDto> findOrderDtos() {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                        " from Order o" +
                        " join o.member m" +
                        " join o.delivery d", OrderSimpleQueryDto.class)
                .getResultList();
    }
}
```
__OrderSimpleQueryDto 리포지토리에서 DTO 직접 조회__
```java
package jpabook.jpashop.repository.order.simplequery;

import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.OrderStatus;
import lombok.Data;

import java.time.LocalDateTime;

@Data
public class OrderSimpleQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate; //주문시간
    private OrderStatus orderStatus;
    private Address address;

    public OrderSimpleQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}
```
- 일반적인 SQL을 사용할 때 처럼 원하는 값을 선택해서 조회
- `new` 명령어를 사용해서 JPQL의 결과를 DTO로 즉시 변환
- SELECT 절에서 원하는 데이터를 직접 선택하므로 DB 애플리케이션 네트웍 용량 최적화(생각보다 미비)
- 리포지토리 재사용성 떨어짐, API 스펙에 맞춘 코드가 리포지토리에 들어가는 단점

#### 정리
엔티티를 DTO로 변환하거나, DTO로 바로 조회하는 두가지 방법은 각각 장단점이 있다. 둘중 상황에
따라서 더 나은 방법을 선택하면 된다. 엔티티로 조회하면 리포지토리 재사용성도 좋고, 개발도 단순해진다. 
따라서 권장하는 방법은 다음과 같다. 

__쿼리 방식 선택 권장 순서__    
1. 우선 엔티티를 DTO로 변환하는 방법을 선택한다.
2. 필요하면 페치 조인으로 성능을 최적화 한다. => 대부분의 성능 이슈가 해결된다.
3. 그래도 안되면 DTO로 직접 조회하는 방법을 사용한다.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접 사용한다.

### API 개발 고급 - 컬렉션 조회 최적화
주문내역에서 추가로 주문한 상품 정보를 추가로 조회하자. Order 기준으로 컬렉션인 `OrderItem` 와 `Item` 이 필요하다.

앞의 예제에서는 toOne(OneToOne, ManyToOne) 관계만 있었다. 이번에는 컬렉션인 일대다 관계(OneToMany)를 조회하고, 최적화하는 방법을 알아보자.

#### 주문 조회 V1: 엔티티 직접 노출
```java
package jpabook.jpashop.api;

import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderItem;
import jpabook.jpashop.domain.OrderStatus;
import jpabook.jpashop.repository.*;
import jpabook.jpashop.repository.order.query.OrderFlatDto;
import jpabook.jpashop.repository.order.query.OrderItemQueryDto;
import jpabook.jpashop.repository.order.query.OrderQueryDto;
import jpabook.jpashop.repository.order.query.OrderQueryRepository;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;
import java.util.List;

import static java.util.stream.Collectors.*;


/**
 * V1. 엔티티 직접 노출
 * - 엔티티가 변하면 API 스펙이 변한다.
 * - 트랜잭션 안에서 지연 로딩 필요
 * - 양방향 연관관계 문제
 *
 * V2. 엔티티를 조회해서 DTO로 변환(fetch join 사용X)
 * - 트랜잭션 안에서 지연 로딩 필요
 * V3. 엔티티를 조회해서 DTO로 변환(fetch join 사용O)
 * - 페이징 시에는 N 부분을 포기해야함(대신에 batch fetch size? 옵션 주면 N -> 1 쿼리로 변경 가능)
 *
 * V4. JPA에서 DTO로 바로 조회, 컬렉션 N 조회 (1 + N Query)
 * - 페이징 가능
 * V5. JPA에서 DTO로 바로 조회, 컬렉션 1 조회 최적화 버전 (1 + 1 Query)
 * - 페이징 가능
 * V6. JPA에서 DTO로 바로 조회, 플랫 데이터(1Query) (1 Query)
 * - 페이징 불가능...
 *
 */
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;
    private final OrderQueryRepository orderQueryRepository;

    /**
     * V1. 엔티티 직접 노출
     * - Hibernate5Module 모듈 등록, LAZY=null 처리
     * - 양방향 관계 문제 발생 -> @JsonIgnore
     */
    @GetMapping("/api/v1/orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAll();
        for (Order order : all) {
            order.getMember().getName(); //Lazy 강제 초기화
            order.getDelivery().getAddress(); //Lazy 강제 초기환
            List<OrderItem> orderItems = order.getOrderItems();
            orderItems.stream().forEach(o -> o.getItem().getName()); //Lazy 강제 초기화
        }
        return all;
    }
}
```
- `orderItem`, `item` 관계를 직접 초기화하면 `Hibernate5Module` 설정에 의해 엔티티를 JSON으로 생성한다.
- 양방향 연관관계면 무한 루프에 걸리지 않게 한곳에 `@JsonIgnore` 를 추가해야 한다.
- 엔티티를 직접 노출하므로 좋은 방법은 아니다.

#### 주문 조회 V2: 엔티티를 DTO로 변환
```java
@GetMapping("/api/v2/orders")
public List<OrderDto> ordersV2() {
    List<Order> orders = orderRepository.findAll();
    List<OrderDto> result = orders.stream()
            .map(o -> new OrderDto(o))
            .collect(toList());

    return result;
}
```
__OrderApiController에 추가__   
```java
@Data
static class OrderDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate; //주문시간
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemDto> orderItems;

    public OrderDto(Order order) {
        orderId = order.getId();
        name = order.getMember().getName();
        orderDate = order.getOrderDate();
        orderStatus = order.getStatus();
        address = order.getDelivery().getAddress();
        orderItems = order.getOrderItems().stream()
                .map(orderItem -> new OrderItemDto(orderItem))
                .collect(toList());
    }
}

@Data
static class OrderItemDto {

    private String itemName;//상품 명
    private int orderPrice; //주문 가격
    private int count;      //주문 수량

    public OrderItemDto(OrderItem orderItem) {
        itemName = orderItem.getItem().getName();
        orderPrice = orderItem.getOrderPrice();
        count = orderItem.getCount();
    }
}
```
- 지연 로딩으로 너무 많은 SQL 실행
- SQL 실행 수
  - `order` 1번
  - `member`, `address` N번(order 조회 수 만큼)
  - `orderItem` N번(order 조회 수 만큼)
  - `item` N번(orderItem 조회 수 만큼)
> 참고: 지연 로딩은 영속성 컨텍스트에 있으면 영속성 컨텍스트에 있는 엔티티를 사용하고 없으면 SQL을
실행한다. 따라서 같은 영속성 컨텍스트에서 이미 로딩한 회원 엔티티를 추가로 조회하면 SQL을 실행하지
않는다.

#### 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화
__OrderApiController에 추가__
```java
@GetMapping("/api/v3/orders")
public List<OrderDto> ordersV3() {
    List<Order> orders = orderRepository.findAllWithItem();
    List<OrderDto> result = orders.stream()
            .map(o -> new OrderDto(o))
            .collect(toList());

    return result;
}
```
__OrderRepository에 추가__
```java
public List<Order> findAllWithItem() {
    return em.createQuery(
            "select distinct o from Order o" +
                    " join fetch o.member m" +
                    " join fetch o.delivery d" +
                    " join fetch o.orderItems oi" +
                    " join fetch oi.item i", Order.class)
            .getResultList();
}
```
- 페치 조인으로 SQL이 1번만 실행됨
- `distinct` 를 사용한 이유는 1대다 조인이 있으므로 데이터베이스 row가 증가한다. 그 결과 같은 order 
엔티티의 조회 수도 증가하게 된다. JPA의 distinct는 SQL에 distinct를 추가하고, 더해서 같은 엔티티가
조회되면, 애플리케이션에서 중복을 걸러준다. 이 예에서 order가 컬렉션 페치 조인 때문에 중복 조회 되는
것을 막아준다.
- 단점
  - 페이징 불가능

> 참고: 컬렉션 페치 조인을 사용하면 페이징이 불가능하다. 하이버네이트는 경고 로그를 남기면서 모든
데이터를 DB에서 읽어오고, 메모리에서 페이징 해버린다(매우 위험하다). 자세한 내용은 자바 ORM 표준
JPA 프로그래밍의 페치 조인 부분을 참고하자.

> 참고: 컬렉션 페치 조인은 1개만 사용할 수 있다. 컬렉션 둘 이상에 페치 조인을 사용하면 안된다. 데이터가
부정합하게 조회될 수 있다. 자세한 내용은 자바 ORM 표준 JPA 프로그래밍을 참고하자.

#### 주문 조회 V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파
