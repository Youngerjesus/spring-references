# Generating Your Own Metadata by Using the Annotation Processor

`spring-boot-configuration-processor` jar 파일을 이용해서 `@ConfigurationProperties` 를 바탕으로 configuration metadata file 을 만들 수 있다.

- 이 jar 파일은 java annotation processor 을 포함하고 프로젝트가 컴파일 될 때 해당 jar 가 실행된다.

## 3.1. Configuring the Annotation Processor

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

gradle 을 쓰고 있다면. 
```gradle
dependencies {
    annotationProcessor "org.springframework.boot:spring-boot-configuration-processor"
}
```

## 3.2. Automatic Metadata Generation


`@ConfigurationProperties` 이 어노테이션이 붙은 클래스나 메소드를 processor 가 pick 한다. 

만약 클래스가 `@ConstructorBinding` 이 있다면 하나의 생성자를 예상함. 


## 3.3. Adding Additional Metadata

