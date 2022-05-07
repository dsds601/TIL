# 인프런 JPA1

* gradle build

* Build 파일에서 ./gradlw build -> build/lib 폴더에 jar 파일 java -jar 자르파일명으로 실행

* 쿼리 파라미터 로그 남기기

* 로깅 버전으로 남기기

  ```yaml
  logging:
    level:
      org.hibernate.SQL: debug
      org.hibernate.type: trace   //<- insert value 가 ? 출력됨
  ```

  * ? 도 value값이 보입니다.
  * implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.6' 라이브러리
  * 운영에서는 성능 저하가 가능성이 있으니 개발 버전에만 사용 권장

* JPA 기본 설정

~~~yaml
spring:
  jpa:
      hibernate:
        ddl-auto: create // jpa 기본 설정 sql문 보이고 ddl create 프로젝트 시작시 마다 테이블이 새로 생성됨
      properties:
        hibernate:
          show_sql: true	// <- sysout 으로 출력됨 운영에서 사용 금지
          format_sql: true

// debug그 레벨로 jap로그 출력
logging:
  level:
    org.hibernate.SQL: debug // <- log로 찍힘
~~~



* test케이스내에 transaction어노테이션이 있으면 테스트 완료후 롤백 한다.

* ```java
  Assertions.assertThat(findMember).isEqualTo(member); // true
  ```

  * 위에 내용이 true인건 같은 영속성 컨텍스트 내부에 동일하게 관리하고 있다면 한 캐시내부에 객체로 참조한다.

~~~java
		@Test
    @Transactional
    @Rollback(false) // <- 테스트 케이스내에 있더라도 rollback 안하고 커밋
    public void testMember () throws Exception{
        //given
        Member member = new Member();
        member.setUsername("memberA");

        //when
        Long id = memberRepository.save(member);
        Member findMember = memberRepository.find(id);
        //then
        Assertions.assertThat(findMember.getId()).isEqualTo(member.getId());
        Assertions.assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
	      Assertions.assertThat(findMember).isEqualTo(member); // true
    }
~~~

