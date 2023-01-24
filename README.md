# 실전 예제 - 1. 요구사항 분석과 기본 매핑

**요구사항 분석**

---

- 회원은 상품을 주문할 수 있다.
- 주문 시 여러 종류의 상품을 선택할 수 있다.

**기능 목록**

---

- 회원 기능
    - 회원 등록
    - 회원 조회
- 상품 기능
    - 상품 등록
    - 상품 수정
    - 상품 조회
- 주문 기능
    - 상품 주문
    - 주문 내역 조회
    - 주문 취소

![img.png](img/img.png)

**도메인 분석 모델**

---

- **회원과 주문의 관계**: **회원**은 여러 번 **주문**할 수 있다. (일대다)
- **주문과 상품의 관계**: **주문**할 때 여러 **상품**을 선택할 수 있다. 반대로 같은 **상품**도 여러 번 **주문**될 수 있다. **주문상품**이라는 모델을 만들어서 다대다 관계를 일대다, 다대다 관계로 풀어냄

![img_1.png](img/img_1.png)

**테이블 설계**

---

![](img/img_2.png)

**엔티티 설계와 매핑**

---

![](img/img_3.png)

*Member*

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String name;

    private String city;

    private String street;

    private String zipcode;

    //Setter, Getter ...
}
```

- 가급적 Setter는 최소한으로 줄여 아무데서나 데이터를 set 할 수 없게 해놓는게 유지보수면에 좋다.
- 테이블 필드의 속성이나 테이블의 제약조건 등 `Entity`에 정의를 해놓으면 개발자가 보기에 편하다.

*Order*

```java
@Entity
@Table(name = "ORDERS")
public class Order {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @Column(name = "MEMBER_ID")
    private Long memberId;

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    //Setter, Getter ...
}

```

*OrderItem*

```java
@Entity
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "ORDER_ITEM_ID")
    private Long id;

    @Column(name = "ORDER_ID")
    private Long orderId;

    @Column(name = "ITEM_ID")
    private Long itemId;

    private int orderPrice;

    private int count;

    //Setter, Getter ...
}

```

*Item*

```java
@Entity
public class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;

    private int price;

    private int stockQuantity;

    //Setter, Getter ...
}

```

*JpaMain*

```java
public class JpaMain {

    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();

        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {
            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }

        emf.close();
    }
}
```

![](img/img_4.png)![](img/img_5.png)![](img/img_6.png)

# 실전 예제 - 2. 연관관계 매핑 시작

**테이블 설계**

---

![](img/img_2.png)

- 실전 예제 1과 테이블 구조는 동일

**객체 구조**

---

- 참조를 사용하도록 변경

![](img/img_7.png)

- **Member 엔티티에 orders(주문 목록)가 존재하는데 잘못된 설계 방식이라고 한다.**
  - 양방향 매핑 예시를 위해 어쩔 수없이 사용함
  - Member 엔티티는 회원에 관련된 멤버 변수만 가지고 있으면 된다.
- **설계할 때는 객체들 간에 매핑을 단방향으로 하고 실제 개발할 때 필요시 양방향으로 적용하는게 좋다.**
- 즉 양방향보단 단방향을 지향한다.

*Member*

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();

    private String name;

    private String city;

    private String street;

    private String zipcode;

    //Getter, Setter...
}
```

- 관례상 멤버 변수가 컬렉션 클래스일때는 new(객체 생성)를 사용해서 초기화를 해준다. 메모리를 좀 쓸 수 있지만 NPE도 방지하고 여러가지 장점이 있다.

*Order*

```java
@Entity
@Table(name = "ORDERS")
public class Order {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<>();

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    public void addOrderItem(OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

		//Getter, Setter...
}
```

*OrderItem*

```java
@Entity
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "ORDER_ITEM_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "ORDER_ID")
    private Order order;

    @ManyToOne
    @JoinColumn(name = "ITEM_ID")
    private Item item;

    private int orderPrice;

    private int count;

    //Getter, Setter...
}
```

*Item*

```java
@Entity
public class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;

    private int price;

    private int stockQuantity;

    //Getter, Setter...
}
```