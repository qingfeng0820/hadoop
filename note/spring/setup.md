# Depends
```
springframeworkVersion = '4.1.5.RELEASE'
springSecurityVersion = '4.0.0.RELEASE'
springDataJpaVersion = '1.7.0.RELEASE'

List springframeworks = [
"org.springframework:spring-core:${springframeworkVersion}",
"org.springframework:spring-context:${springframeworkVersion}",
"org.springframework:spring-context-support:${springframeworkVersion}",
"org.springframework:spring-jdbc:${springframeworkVersion}",
"org.springframework:spring-beans:${springframeworkVersion}",
"org.springframework:spring-webmvc:${springframeworkVersion}",
"org.springframework:spring-tx:${springframeworkVersion}",
"org.springframework:spring-orm:${springframeworkVersion}",
"org.springframework:spring-aop:${springframeworkVersion}",
"org.springframework:spring-aspects:${springframeworkVersion}",
"org.springframework:spring-expression:${springframeworkVersion}",
"org.springframework:spring-test:${springframeworkVersion}"
]

List springSecurity = [
"org.springframework.security:spring-security-web:${springSecurityVersion}"
"org.springframework.security:spring-security-config:${springSecurityVersion}"
"org.springframework.security:spring-security-core:${springSecurityVersion}"
]

compile springframeworks
compile springSecurity
compile "org.springframework.data:spring-data-jpa:${springDataJpaVersion}"
```
