# 자동 구성 라이브러리 사용하기2

## 라이브러리 추가

`project-v2/libs` 폴더를 생성하자.

`memory-v2` 프로젝트에서 빌드한 `memory-v2.jar` 를 이곳에 복사하자. 

`project-v2/build.gradle` 에 `memory-v2.jar` 를 추가하자.

```groovy
dependencies {
    implementation files ('libs/memory-v2.jar')
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

```

## 라이브러리 설정

앞서 `project-v1` 에서는 `memory-v1` 을 사용하기 위해 스프링 빈을 직접 등록했다.

`project-v2` 에서 사용하는 `memory-v2` 라이브러리에는 스프링 부트 자동 구성이 적용되어 있다. 

따라서 빈을 등록하는 별도의 설정을 하지 않아도 된다.


메모리 조회 기능이 잘 동작하는지 확인해보자.

**서버 실행 로그** 

```
MemoryFinder : init memoryFinder
```

**실행**

http://localhost:8080/memory

**결과** 

```json
{"used": 38174528, "max": 8589934592} 
```

메모리 조회 라이브러리가 잘 동작하는 것을 확인할 수 있다.

`memory=on` 조건을 끄면 라이브러리를 사용하지 않는 것도 확인할 수 있다.

**정리**

스프링 부트가 제공하는 자동 구성 덕분에 복잡한 빈 등록이나 추가 설정 없이 단순하게 라이브러리의 추가 만으로 프로젝트를 편리하게 구성할 수 있다.

`@ConditionalOnXxx` 덕분에 라이브러리 설정을 유연하게 제공할 수 있다.

스프링 부트는 수 많은 자동 구성을 제공한다. 

그 덕분에 스프링 라이브러리를 포함해서 수 많은 라이브러리를 편리하게 사용할 수 있다.
