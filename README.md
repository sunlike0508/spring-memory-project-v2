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

# 자동 구성 이해 1 - 스프링 부트의 동작

스프링 부트는 다음 경로에 있는 파일을 읽어서 스프링 부트 자동 구성으로 사용한다. 

`resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

우리가 직접 만든 `memory-v2` 라이브러리와 스프링 부트가 제공하는 `spring-boot-autoconfigure` 라이브러 리의 다음 파일을 확인해보면 스프링 부트 자동 구성을 확인할 수 있다.

**memory-v2 - org.springframework.boot.autoconfigure.AutoConfiguration.imports**

```
 memory.MemoryAutoConfig
```

이번에는 실제 스프링 부트를 보면

**spring-boot-autoconfigure - org.springframework.boot.autoconfigure.AutoConfiguration.imports**

org.springframework.boot/spring-boot-autoconfigure/3.0.2/42ad589ec930e05a2ed702a4940955ff97b16a8c/spring-boot-autoconfigure-3.0.2.jar!/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports

```properties
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration
org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration
...
```

이번에는 스프링 부트가 어떤 방법으로 해당 파일들을 읽어서 동작하는지 알아보자. 

이해를 돕기 위해 앞서 개발한 `autoconfig` 프로젝트를 열어보자.

스프링 부트 자동 구성이 동작하는 원리는 다음 순서로 확인할 수 있다. 

`@SpringBootApplication` -> `@EnableAutoConfiguration` -> `@Import(AutoConfigurationImportSelector.class)`

스프링 부트는 보통 다음과 같은 방법으로 실행한다. **AutoConfigApplication**

```java
@SpringBootApplication
public class AutoConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(AutoConfigApplication.class, args);
    }
}
```
`run()` 에 보면 `AutoConfigApplication.class` 를 넘겨주는데, 이 클래스를 설정 정보로 사용한다는 뜻이다. 

`AutoConfigApplication` 에는 `@SpringBootApplication` 애노테이션이 있는데, 여기에 중요한 설정 정보들이 들어있다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
}
```

여기서 우리가 주목할 애노테이션은 `@EnableAutoConfiguration` 이다. 이름 그대로 자동 구성을 활 성화 하는 기능을 제공한다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
}
```

`@Import` 는 주로 스프링 설정 정보( `@Configuration` )를 포함할 때 사용한다.

그런데 `AutoConfigurationImportSelector` 를 열어보면 `@Configuration` 이 아니다.

이 기능을 이해하려면 `ImportSelector` 에 대해 알아야 한다.

# 자동 구성 이해2 - ImportSelector

`@Import` 에 설정 정보를 추가하는 방법은 2가지가 있다.

* 정적인 방법: `@Import` (클래스) 이것은 정적이다. 코드에 대상이 딱 박혀 있다. 설정으로 사용할 대상을 동적으로 변경할 수 없다.
* 동적인 방법: `@Import` ( `ImportSelector` ) 코드로 프로그래밍해서 설정으로 사용할 대상을 동적으로 선택할 수 있다.

**정적인 방법**

스프링에서 다른 설정 정보를 추가하고 싶으면 다음과 같이 `@Import` 를 사용하면 된다. 

```java
 @Configuration
 @Import({AConfig.class, BConfig.class})
public class AppConfig {...} 
```

그런데 예제처럼 `AConfig` , `BConfig` 가 코드에 딱 정해진 것이 아니라, 특정 조건에 따라서 설정 정보를 선택해야 하는 경우에는 어떻게 해야할까?

**동적인 방법**

스프링은 설정 정보 대상을 동적으로 선택할 수 있는 `ImportSelector` 인터페이스를 제공한다.


## ImportSelector

```java
package org.springframework.context.annotation;
public interface ImportSelector {

 String[] selectImports(AnnotationMetadata importingClassMetadata);
 //...
}
```

이해를 돕기 위해 간단하게 `ImportSelector` 를 사용하는 예제를 만들어보자.

ImportSelector 예제

다음 예제들은 모두 `src/test` 하위에 만들자 

**HelloBean**

```java
package hello.selector;
public class HelloBean {
}
```
빈으로 등록할 대상이다.