# JPA 매핑

* **컨트롤러에서 Entity를 반환하면 안됩니다. 무한루프가 생길수 있고 실수로 API에서 엔티티를 변경할경우 테이블이 변경될수도 있습니다.**



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

      

* ### 연관관계 매핑

* 객체와 테이블의 연관관계의 차이를 이해 , 객체의 참조와 외래키 매핑 차이를 이해

  * 외래키를 객체의 주소를 가져옴니다.
  * 방향 : 단방향 , 양방향
  * 다중성 : N:1 ,1:N , N:M
    * 회원과 팀 -> N:1 다대일
  * 연관관계의 주인 : 객체는 양방향 연관관계와 달리 관계의 주인이 필요함

* **ORM 매핑시 외래키를 객체의 주소가 아닌 키값으로 매핑할때 단점**

  * 객체를 조회올때 객체의 주소로 바로 참조하는게 아니라 영속성컨텍스트 내부에 키값을 조회해 다시 찾아야한다.
    * **객체지향스럽지않다....** -> 객체의 주소를 참조하여 외래키값이 객체의 주소라 그 객체를 가져오자

  ~~~java
  Member findMember = em.find(Member.class,member.getId());
  Team findTeam = em.find(Team.class,findMember.getTeamId());
  ~~~

  * 객체지향스러운 모델링
  * Member 엔티티에 팀 객체의 주소를 가지고 있어 바로 조회가 가능
  * **단방향일경우**

  ~~~sql
  N : 1
  Member							Team
  Team team						id
  id									name
  userName
  ~~~

  * 멤버가 다 (N)이고 팀이 1이라면 Member Entity에는 team 을 @ManyToOne이라는 어노테이션을 작성하고 ->외래키 매핑을위해 어떤컬럼을 조인할지 어노테이션 @JoinColumn을 작성하고 컬럼에 name값을 적는다

  ~~~java
  @Entity
  class Member {
  	@ManyToOne // member가 다 팀이 1   관계가 어떤지
  	@JoinColumn(name ="TEAM_ID") // 외래키 이름	조인할때에 컬럼이름
  	Team team;
  }
  
  // 매핑할 객체 값을 넣을때
  member.setTeam(team) //<-바로 넣으면 됩니다. 변경도 동일하게 다른팀 객체를 넣으면 외래키 값이 업데이트되어 연관관계를 수정가능합니다.
  ~~~

  

* ### 연관관계의 주인

* **단방향 매핑으로 끝내는게좋다**

* 양방향 매핑 ( 객체는 단방향이 좋고 양방향일 경우 중간 테이블 만드는게 좋을 가능성이 높습니다.)

~~~
N : 1
Member							Team
Team team						id
id									name
userName						List members
~~~

* 테이블에 연관관계는 키값하나로 양방향을 알 수 있기에 양방향연관관계라는 개념이없다
* 하지만 객체는 서로 주소를 알고 있어야 서로 참조할수있기에 양방향의 관계가 중요하다.

~~~java
@Entity
class Team {
  // 어떤 변수와 매핑되어있는지 연관관계가 있는 객체의 키값의 변수명을 작성한다 여기서는 Member에 team Id 조인한 컬럼인 team 변수이다.
  @OneToMany(mappedBy = "team") 
	priate List<Member> members = new ArrayList();
}

~~~

* mappedBy : 어떤 변수에 매핑되어있는지 알려주는 속성

* ### 연관관계의 주인 mappedBy

* **주인은 외래키가 있는곳을 주인으로 지정!!!** (데이터베이스입장에서 생각하자)

* ex) 외래키가 없는곳이 주인이라면 Team클래스에서 members에 객체를 변경했을경우 TEAM 테이블에서 변경을 했는데 MEMBER 테이블에 update 쿼리가 나가는게 이상하다.

* 외래키가 있는곳은 무조건 N이다.

* 무조건 N쪽이 연관관계의 주인이 된다.(컬렉션을 들고있는 엔티티는 연관관계의 주인이 아니다.)

  * 객체와 테이블간에 연관관계를 맺는 차이를 이해해야합니다.

  * 객체 연관관계 2개

    * 회원 -> 팀 (단방향) / 팀 -> 회원 (단방향)
      * 사실 양방향이 아니라 단방향이 2개인것 =>양방향 위해서는 단방향 2개를 만들어야한다.

  * 테이블 연관관계 1개

    * 팀에 아이디가 멤버에 외래키하나로  양쪽에 관계를 알 수 있다.

  * **양방향 연관관계일 경우 외래키 관리**

    * 멤버에도 팀에 키값이 있고 팀에도 키값이 있다. 멤버를 수정하고 싶을때 팀에있는 members에 값을 바꿀지 Member에 있는 값을 바꿔야할지 애매하다. 디비 입장에서는 멤버에 있는 팀키값만 변경되면 됩니다.
    * 애매하기에 둘중 하나를 주인을 정해 관리하게 됩니다. => **연관관계의 주인**

  * ### 양방향 매핑 규칙

    * **주인은 외래키가 있는곳을 주인으로 지정!!! **(데이터베이스입장에서 생각하자)

    * 객체의 두 관계중 하나를 연관관계 주인으로 지정

    * 주인만 외래키 관리(등록,수정)

    * 주인이 아닌쪽 읽기만 가능

    * **주인은 mapppedBy속성 하지 않는다. 주인이 아니면 mappedBy속성으로 주인을 지정한다.**

      * @OneToMany(mappedBy = "")
      * mappedBy ~~에 당한거니 주인이 아닌경우라 생각
  
    * 위 예제에서는 Member.team이 연관관계의 주인이다 **무조건 N쪽이 연관관계의 주인이 된다.**
  
    * 연관관계의 주인이 아닌쪽에 값을 넣으면 외래키값이 null이 된다.
  
    * **순수 객체 상태를 고려해야하기에 양쪽에 값을 넣으면 됩니다.**
  
      * 테스트및 영속성 컨텍스트에서 플러시가 안됫을경우 디비에 값을 조회해 오지 않기에 1차캐시내에 값이 컬렉션에 들어 있지 않아서 JPA가 못읽어오는 경우가 있습니다.
      * 양쪽에 값을 넣는경우 잊어 먹을수 있기에 연관관계 편의 메서드를 작성하자.
      * 편의메서드는 주인이든 주인이아니든 연관관계 편의메서드를 작성하여 사용하면 된다.
  
      ~~~java
      class Member {
      	@ManyToOne
      	@JoinColumn("")
      	Team team;
      	
      	public void changeTeam(Team team) {
      		this.team= team;
      		team.getMembers.add(this); //<- 이렇게 편의메서드를 작성하면 연관관계의 주인이 아닌 Team엔티티에서 굳이 엔티티를 넣지 않아도 된다. 메서드명도 setTeam 등 처럼 작성하면 get / set관례처럼 보일수 있기에 changeTeam 등 네이밍을 변경하여 메서드명을 작성하는게 좋다
      	}
      }
      ~~~
  
  * 양방향 매핑시 무한루프를 조심하자
    * toString() <- 일때 A에 toString일때 members등 조인할 컬럼을 호출할 경우 다른 엔티티를 호출하며 서로가 서로를 호출하여 무한루프에 빠질수있다.
    * **주의 !! : lombok에 toString() 사용시 변수를 다 잡아서 주의 JSON생성 라이브러리도 마찬가지**

* ### 연관관계 매핑

  * 단방향 , 양방향
    * 테이블 : 외래키 하나로 양쪽 조인 가능 방향이라는 개념이 필요 없음
    * 객체 : 참조용 필드가 있는 쪽으로만 참조 가능 단방향 양방향 개념이 있음
  *  연관관계의 주인
  * 테이블은 외래키 하나로 두테이블이 연관관계를 맺음
  * 객체 양방향 관계는 A -> B B->A 처럼 참조가 2군데 둘중 **테이블의 외래키 관리할 곳을 지정해야함**
  * **연관관계의 주인** : 외래키를 관리하는 참조
    * **주인은 외래키가 있는곳을 주인으로 지정!!! **(데이터베이스입장에서 생각하자)
    * 다쪽 이 외래키가 있어야한다. 다쪽이 주인이다.
  * 주인이 아니면 외래키에 영향을 주지않고 단순 조회만 가능
    * @OneToMany(mappedBy ="") **다대일 다 가 연관관계의주인일 경우**
  * **일대 다 : 일이 연관관계의 주인일때 권장하지않음**
    * 일이 외래키를 관리 할 때 (권장하지 않음)
    * 1인 테이블이 반대편 테이블의 외래키를 관리하는 특이한구조
    * 연관관계 관리를 위해 추가로 UPDATE 실행
    * @JoinColumn 사용하지않으면 조인테이블(@JoinTable) 쿼리를 사용하게 됩니다. (중간 테이블 쿼리) -> 성능상 단점 운영상 테이블이 많아 헷갈림
    * 차라리 다대일 양방향을 사용
  * **일대일: 반대도 일대일**
    * 주 테이블 대상 테이블 모두 외래키 선택 가능
    * 외래 키에 데이터베이스 유니크 제약 조건 추가
    * 다대일 단방향 매핑이랑 구조가 비슷함
      * 양방향을 원할 경우 mappedBy 지정필수
    * 일대일 정리
      * 주 테이블에 외래키
        * 주 객체가 대상 객체의 참조를 가지는 것
        * 객체지향 개발자 선호
        * JPA 매핑 편리
        * 장점 : 주 테이블 조회로 대상 테이블 데이터 확인 가능
        * 단점 : 값이 없을때 외래키에 null 허용
      * 대상 테이블에 외래키
        * 대상 테이블에 외래 키가 존재
        * 전통적인 데이터베이스 개발자 선호
        * 장점 : 주 테이블과 대상 테이블 일대일에서 일대다 관계로 변경시 테이블 구조 유리
        * 단점 : 프록시 기능한계로 지연로딩해도 즉시로딩이 됨 -> 대상 테이블을 찾아야하기때문에 어차피 조회 쿼리가 나가기에 즉시 로딩이 됨
  * **다대다 : 사용하면 안됩니다. (테이블 새로 만들어 사용합니다 -> @JoinTable x )**
    * 관계형 데이터 베이스는 테이블 2개로 다대다 관계가 불가능합니다.
    * 중간 테이블을 구성하여 일대다 다대일 관계로 풀어야합니다.
    * **객체는 컬렉션을 사용해서 객체 2개로 다대다 관계가 가능합니다.**
    * @ManyToMany @JoinTable 사용하여 만들수 있습니다.
    * 연결테이블이 단순 연결만 하고 끝나지 않습니다. 주문시간 수량 같은 데이터가 추가로 들어올수 있음 하지만 중간 테이블 만들경우 추가가 안됩니다. 중간 테이블이 숨겨져 있어 쿼리도 이상하게 나옵니다.
    * **해결법 : ** 연결 테이블용 엔티티를 추가로 만듭니다.(연결테이블 -> 엔티티)
      * @ManyToMany -> @OneToMany / @ManyToOne
  
  #### @ManyToOne
  
  * 다대일 관계 매핑
  * 다대일은 mappedBy가 없다 -> **다대일 경우에는 연관관계의 주인이 되어야한다!!**
  
  | 속성         | 설명                                                         | 기본값                                     |
  | ------------ | ------------------------------------------------------------ | ------------------------------------------ |
  | optional     | false로 설정하면 연관된 엔티티 필수                          | true                                       |
  | fetch        | 글로벌 페치 전략 설정                                        | @ManyTonOne = EAGER,<br>@OneTonMany = LAZY |
  | cascade      | 영속성 전이 기능 사용                                        |                                            |
  | targetEntity | 연관된 엔티티 정보 설정 (제네릭이 생경 잘 사용하지 않습니다.) |                                            |

#### @OneToMany

* 일대다 관계 매핑

  | 속성         | 설명                                                         | 기본값 |
  | ------------ | ------------------------------------------------------------ | ------ |
  | mappedBy     | 연관관계의 주인 필드 선택                                    |        |
  | fetch        | 글로벌 페치 전략설정                                         |        |
  | cascade      | 영속성 전이 기능 사용                                        |        |
  | targetEntity | 연관된 엔티티 정보 설정 (제네릭이 생경 잘 사용하지 않습니다.) |        |

  

* ### 상속관계 매핑

  * 관계형 데이터베이스는 상속관계가 없다 객체에 상속관계가 비슷한 구조는 슈퍼타입 서브타입이라는 모델링 기법이랑 유사
  * 상속관계 매핑이라는것은 DB의 슈퍼타입 서브타입 관계를 매핑
  * 디비로 상속관계 전략으로는 조인전략 , 단일 테이블 전략 , 구현 클래스마다 테이블 전략
  * **조인 전략** :상속과 제일 비슷한듯 상위에 추상클래스처럼 상위 테이블을 두고 나머지가 fk를 가져가는 방식
    * 정규화 되어있다. , 기본적으로 이 전략을 사용한다.
    * 객체 상속과 동일하게 Album에 상속한 값을 넣으면 상속한 테이블에 값이 들어가게된다.
    * select시 조인해서 가져오게된다.

  ~~~java
  @Entity
  @Inheritance(strategy = InheritanceType.JOINED) // 조인 전략 default 단일 테이블 전략
  @DiscriminatorColumn // <- Dtype으로 값이 들어올경우 Entity명이 들어가게된다. (왠만하면 넣는게 좋다)
  public class Item {
  	String	name;
  }
  @Entity
  @DiscriminatorValue("A") // <- 상위 테이블에 들어가는 Dtype 컬럼에 값을 변경할 경우
  public class Album extends Item{
  
  }
  @Entity
  public class Movie extends Item{
  
  }
  ~~~

  

  * **싱글 테이블전략 ** : 단일 테이블로 하나에 테이블에 다 들어가게되고 DTYPE으로 어떤 테이블에 값이 들어오는지 체크

    * 위와 동일하고 @Inheritance(strategy = InheritanceType.SINGLE_TABLE) 변경 조인이 아니고 한 테이블에 다들어가게됨
    * 조인할 필요도 없기에 성능상 이점이 있다. DTYPE당연 필수 @DiscriminatorColumn 없어도 값이 무조건 들어감

  * **구현 클래스마다 테이블 전략** : 모두 각 테이블마다 상속한 테이블에 키값이 들어가고 상속한 엔티티에 값이 상속받은 엔티티에 들어가게된다.

    * **사용하면 안됩니다.**
    * @DiscriminatorColumn 의미가 없다 각각 테이블로 조회를 하기에 DTYPE이 필요가없다
    * 조회를 할때 모든 엔티티를 유니온으로 묶어서 한번에 조회를 하게 된다.
      * 성능상 효율이 좋지 않다.

    ~~~java
    @Entity
    @Inheritance(strategy = InheritanceType.PER_TABLE) // 조인 전략 default 단일 테이블 전략
    public abstract class Item {
    	String	name;
    }
    @Entity
    @DiscriminatorValue("A") // <- 상위 테이블에 들어가는 Dtype 컬럼에 값을 변경할 경우
    public class Album extends Item{
    
    }
    @Entity
    public class Movie extends Item{
    
    }
    ~~~

  

* ### @MappedSuperclass

  * **상속 관계 매핑이 아닙니다.**
  * **해당 클래스는 엔티티가 아닙니다 당연히 조회와 검색이 안됩니다.**
  * 직접 사용할 일이 없기에 추상클래스로 만드는걸 권장합니다.
  * 공통 매핑 정보가 필요할 때 사용 -> 상속이랑 상관이 없습니다.
    * ex) A 테이블에 name age가 들어가고 B테이블에도 name age가 들어갈 경우 사용 , 모든 테이블에 날짜가 들어가는 경우

  ~~~java
  @MappedSuperclass // <- 매핑 정보만 받는 클래스라고 명시
  abstract class BaseEntity{
  	private String createdBy;
  }
  
  @Entity
  class Member extends BaseEntity{
  
  }
  ~~~

  
