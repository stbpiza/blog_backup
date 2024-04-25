# [Spring Boot] JPA 필터 정렬 구현

![](https://images.velog.io/images/stbpiza/post/1a950943-ee0c-4f9e-8fb6-1a9958c18a89/springboot.png)


# JPA 필터 정렬 구현
## 사용한 툴
> - spring-boot 2.6.3
- spring-boot-starter-data-jpa
- querydsl 1.0.10

## 환경세팅
### build.gradle
```json
buildscript {
    ext {
        queryDslVersion = "5.0.0"
    }
}

plugins {
    id 'org.springframework.boot' version '2.6.3'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
    implementation "com.querydsl:querydsl-apt:${queryDslVersion}"

    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'mysql:mysql-connector-java'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}

def querydslDir = "$buildDir/generated/querydsl"
querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}
sourceSets {
    main.java.srcDir querydslDir
}
configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
    querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
```


### application.yml
```yaml
server:
  port: 8000

spring:

  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/filterdemo?serverTimezone=Asia/Seoul
    username: root
    password: root1234
    hikari:
      maximum-pool-size: 20

  jpa:
    open-in-view: false
    hibernate:
      ddl-auto: update
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
      use-new-id-generator-mappings: false
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        default_batch_fetch_size: 1000
```
### 구조
![](https://images.velog.io/images/stbpiza/post/55d54687-16b2-4725-a659-4a0d673bf0a6/image.png)

테스트 더미 데이터를 자동으로 넣어주는 GeneratorService 생성

> ![](https://images.velog.io/images/stbpiza/post/2c6bc99b-e317-41fe-b3f8-ccd0a622d1fb/image.png)
필터와 정렬에 사용할 DB 제작
user - 유저 테이블
board - 게시물 테이블(postType로 필터 구현, 작성시간과 조회수로 정렬 구현)
language - 프로그래밍 언어 테이블(언어 종류로 필터 구현 + 계속 확장할 수 있도록 테이블을 따로 뺌)
languagemapping - 프로그래밍 언어와 게시물 다대다 매핑 테이블

## 코드
### DTO
> 필터와 정렬에 사용할 DTO
![](https://images.velog.io/images/stbpiza/post/08653952-d549-4c9b-8d1d-990b900dbd1f/image.png)
쿼리스트링에 사용
Setter를 꼭 만들어줘야함
![](https://images.velog.io/images/stbpiza/post/64bbfbed-7c5e-4d4f-9cfa-cb425f538f36/image.png)
![](https://images.velog.io/images/stbpiza/post/b8a42d3e-9edd-4210-94f8-826b6111f52e/image.png)
PostType과 BoardOrderType은 enum으로 만들기
enum으로 만들면 스웨거에 리스트가 보여서 프론트가 보기 편함 + 오타발생시 예외발생

> 게시물 리스트 DTO
![](https://images.velog.io/images/stbpiza/post/06cbc907-e8c6-4de6-9ffd-6d5a7f1d5cd0/image.png)
repository에서 List< Board >로 가져와서 생성자에서 DTO에 담기

> 게시물 리스트에 들어가는 세부 DTO
![](https://images.velog.io/images/stbpiza/post/c4adee7d-6470-408d-b101-9e995a88db63/image.png)
생성자 인풋에 Board 하나만 두면 stream().map(BoardListDto::new) 처럼 깔끔하게 작성 가능
다만 board.get으로 다른 entity 가져오다가 nullPointerException 발생가능

### Repository

> 게시물 repository
![](https://images.velog.io/images/stbpiza/post/9f5b117b-f4d0-4f56-9a28-fc75be8306ed/image.png)
![](https://images.velog.io/images/stbpiza/post/e45c5fbd-a03c-4e38-b52d-0a6e54948c20/image.png)
where 문에는 BooleanExpression으로 리턴해서 넣기
(null이 들어가면 없는 조건으로 처리됨)
order 문에는 OrderSpecifier로 리턴해서 넣기

### Service

> 게시물 service
![](https://images.velog.io/images/stbpiza/post/1e13339a-dcdb-493e-bcff-b7c00e686a78/image.png)

### Controller
> 게시물 controller
![](https://images.velog.io/images/stbpiza/post/8a7882d0-3497-4e62-94c5-39fbc252efd5/image.png)
인풋에 @RequestBody 없이 dto 넣으면 쿼리스트링으로 들어감

## 결과 조회

> board.http
![](https://images.velog.io/images/stbpiza/post/78784684-eef1-4bb8-95a2-330d21a15109/image.png)
http 파일로 포스트맨처럼 api 요청 가능
![](https://images.velog.io/images/stbpiza/post/be0991d6-fa98-4d27-ac7b-61cfe168fdf7/image.png)
적용이 잘 되는 모습
![](https://images.velog.io/images/stbpiza/post/a9c0a615-4a93-40e3-98c1-2555ecaadee3/image.png)
repository에서 null 예외처리를 다 해놔서 쿼리스트링 빼먹어도 작동 잘 됨
잘못된 값 입력시 enum 에러발생하여 400으로 응답

## 마무리

모든 코드는 git에 있습니다.
https://github.com/stbpiza/filterDemo

김영한님의 JPA 강의를 많이 참고하였습니다.
감사합니다.