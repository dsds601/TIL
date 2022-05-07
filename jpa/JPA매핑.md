# JPA 매핑

* ### @Entity

  * JPA가 관리하는 클래스
  * 주의 기본생성자 필수 -> 동적으로 리플렉션 , 프록시등등 사용하기에
  * Final 클래스 enum , interface , inner 클래스 사용 x
  * 디비 저장할 필드에 final사용 x

* @Table : 엔티티와 매핑할 테이블 지정

  * 속성은 검색 -> name , catalog , schema , uniqueConstraints(DDL 생성시 유니크 제약조건 생성) ...

* 데이터베이스 스키마 자동생성

  * DDL을 애플리케이션 실행시점에 자동생성 -> 개발단계에 사용 **ddl.auto : create ,update ,validate ...**

* ### @Column

  * **주의 컬럼에 unique제약 조건을 걸수 있지만 컬럼명이 랜덤값으로 들어와 어떤값인지 구분을 못해 @Table에서 제약조건을 준다**
  * columnDefinition : 데이터베이스 컬럼 정보를 직접줄수 있음
    * varchar(100) default 'EMPTY'
  * Precision,scale(DDL) :Bigdecimal 혹은 BigInteger에서 사용
  * enum타입을 사용하고 싶다면 @Enumerated(EnumType.String)
    * **주의 default가 ORDINAL** String 권장 아니면 enum 순서를 데이터베이스에 저장 되어 구분하기 힘들다.
  * @Temporal(Data / Time / TimeStamp) :날짜 타입
    * LocalDateTime ,LocalDate 타입 으로 사용가능  **굳이 @Temporal 필요없음**
  * @Lob :varchar를 넘는 많은 데이터
  * @Transient : 디비와 관계없이 메모리에서 계산을 위해 엔티티객체에 넣는경우

* ### 기본키 매핑

  * @ID : 직접할당

  * @GeneratedValue (자동생성)

    * AUTO : DB 방언에 맞는 자동생성
    * IDENTITY : 기본키 생성 데이터베이스에 위임 ex) auto_Increment
    * SEQUENCE :  @SequenceGenerate로 시퀀스 생성해 이름과 값 생성 가능
      * @Entity에 시퀀스 생성
    * TABLE : 키 생성 전용 테이블을 만들어 데이터베이스 시퀀스 흉내내는 전략
      * @Entity에 @TableGenerator 생성
      * 모든 데이터베이스 사용 가능하나 단점으로 성능이 떨어진다.

  * 권장하는 식별자 전략

    * **비즈니스 컬럼을 PK로 사용하지 말자!**
    * Long형 + 대체키 + 키 생성전략 사용 || Autoincrement || Sequence

  * IDENTITY전략 특징

    * 데이터베이스에 값이 들어가야 조회가 가능 영속성 컨텍스트내부에서 디비에 들어가기전에는 알 수 없어서 **Identity전략으로 사용할땐 persist호출 시점에 바로 쿼리가 날아가게된다.**
    * 영속성 컨텍스트에 관리 되는 시점에 디비가 들어가있게된다.
    * 단점 : 데이터를 모아서 쿼리를 한번에 보내는게 불가능하다. -> 크게 성능에 차이가 나지 않는다.

  * SEQUENCE 전략 특징

    * IDENTITY와 마찬가지로 영속성 컨텍스트에서 키값을 알 수 있어야하기에 persist할때 디비에 가서 조회를 하고 온다.

      * 네트워크를 왓다갓다 하기에 성능상 문제가 있을수있어 allocateSize를 통해 최적화가 가능하다.
      * **미리 데이터를 시퀀스 값을 디비에 올려놓고 메모리에서 가져온 개수 만큼 사용하는 방식** -> 데이터를 가져온 후 부터는 메모리에서 사용한다.

      ~~~java
      @SequenceGenerator(
              name = "MEMBER_SEQ_GENERATOR",
              sequenceName = "MEMBER_SEQ", //데이터베이스에 저장할 시퀀스 이름
              initialValue = 1,allocationSize = 50
      )
      public class Member {
        
          @Id 
        	@GeneratedValue(strategy = GenerationType.SEQUENCE,
          generator = "MEMBER_SEQ_GENERATOR")
          private Long id;
      ~~~

      

