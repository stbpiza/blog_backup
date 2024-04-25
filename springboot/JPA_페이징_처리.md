# [Spring Boot] JPA 페이징 처리

![](https://images.velog.io/images/stbpiza/post/d4ca1cb9-ab57-4962-be97-f58676c3f17a/springboot.png)
# JPA 페이징 처리
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
    url: jdbc:mysql://localhost:3306/pagedemo?serverTimezone=Asia/Seoul
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
![](https://images.velog.io/images/stbpiza/post/88791adf-2fd7-4ba8-9d21-65c8140506a0/image.png)

## 코드
### Entity
> 게시물 엔티티
![](https://images.velog.io/images/stbpiza/post/819c376d-3385-4035-9734-db239c086183/image.png)

> 유저 엔티티
![](https://images.velog.io/images/stbpiza/post/d2458b11-a3fe-40d9-8f94-27950bae0cef/image.png)
유저와 게시물과는 1대다로 연결
유저 닉네임과 이메일은 유니크 설정

### DTO
> 게시물리스트 DTO
![](https://images.velog.io/images/stbpiza/post/8c7db4dd-a438-4608-bd2e-2b48ed4c8f05/image.png)
생성자에서 엔티티 리스트를 DTO 리스트로 옮김
총 페이지와 총 게시물 제공

> 게시물리스트에 들어갈 세부 DTO
![](https://images.velog.io/images/stbpiza/post/7eeef7ff-4a68-40d9-a246-68715cd2eba7/image.png)
리스트에 보여줄 항목들만 넣음
제목, 조회수, 작성시간, 작성자 닉네임

### Repository
> 게시물 repository
![](https://images.velog.io/images/stbpiza/post/1df3ed08-33e1-4e7b-b118-d95a6a1f5c54/image.png)
페이징 처리는 Pageable로 구현
fetchJoin으로 유저 정보까지 한 번에 조회
orderBy는 일단 boardId 순으로 설정
return은 Page< > 형식

### Service
> 게시물 service
![](https://images.velog.io/images/stbpiza/post/dfffafe8-5583-4615-a8d7-330fc2ee2085/image.png)
Page< > 형식으로 DB에서 꺼낸 후 DTO에 Builder로 넣음
getContent()로 List< Board > 꺼내기
getTotalElements()로 총 게시물 수 꺼내기(long)
getTotalPages()로 총 페이지 수 꺼내기(int)
ResponseEntity< >로 상태코드까지 리턴

### Controller
> 컨트롤러
![](https://images.velog.io/images/stbpiza/post/aab24f46-2a72-43fb-879d-4bfd1cdcf0c6/image.png)
RestController로 데이터만 리턴
ResponseEntity< >로 리턴


## 결과 조회
> board.http
![](https://images.velog.io/images/stbpiza/post/ac7e51f0-2926-4e42-88bc-0aabb6ce2cc0/image.png)
.http 파일을 만들면 포스트맨처럼 api 요청 가능
![](https://images.velog.io/images/stbpiza/post/b55c1e33-0411-45be-bda8-c96e0331ccd7/image.png)
![](https://images.velog.io/images/stbpiza/post/cf38c760-03b2-455d-a0d1-386e8739d8f6/image.png)
아무런 값 없이 조회하면 size=20, page=0 으로 요청

> size=10, page=2 로 조회
![](https://images.velog.io/images/stbpiza/post/f487c283-f33b-4f57-a779-a4f0ad5a03c6/image.png)
![](https://images.velog.io/images/stbpiza/post/957473d0-7de3-4c07-8f3f-ffe33beb7b3b/image.png)
![](https://images.velog.io/images/stbpiza/post/b63376d8-a348-4e15-bac1-aa10d970794c/image.png)
총 페이지수가 4에서 7로 늘어나는걸 보면 size 적용이 잘 된 모습
page는 0부터 시작하기에 2를 넣자 3페이지가 보임

## 마무리

모든 코드는 git에 있습니다.
https://github.com/stbpiza/pageDemo

김영한님의 JPA 강의를 많이 참고하였습니다.
감사합니다.