---
title: " 🛩️ Monolithic to Multi-Module Architecture "
layout: single
classes: wide
categories:
  - Architecture
  - SideProject
---

## 🚛 멀티 모듈 구조로 이사가기!
 
### 1. 기존 모놀리틱(Monolithic) 구조에서 느낀 문제점

  작년 10월부터 진행한 사이드 프로젝트인 대피로(Daepiro) 백엔드는 모놀리틱 구조로 출발했습니다. 2개월 내에 데모 어플리케이션을 빠르게 개발해야 하는 촉박했던 일정과, 리소스가 적었기 때문에 간단한 모놀리틱 구조를 선택했었죠. 하지만 개발이 점차 진행될수록 여러가지 문제점이 발생했습니다.

1. 기능 변경/추가 시 영향도를 예측할 수 없다.
2. 관리 포인트가 늘어난다.
   - 단일 모듈에서 모든 기능을 관리하므로 복잡도가 높아지고, 확장성이 크게 떨어지고 있었습니다.
3. 코드 중복이 자주 발생한다.

  이 문제를 해결하기 위해 고려한 선택지는 두 가지가 있었습니다. 기존 구조를 유지한 상태에서, 전반적으로 리팩토링을 시도하거나 새로운 아키텍쳐로의 전환을 모색하는 것 이었죠. 첫 번째 방법은 단기간 내에 문제를 해결할 수 있는 매력적인 선택지이었지만, 근본적인 문제점을 해결하지는 못할 것이라는 우려가 있었습니다. 결국 모놀리틱 구조에서 탈피하여, 확장성과 유지보수성을 확보할 수 있는 새로운 구조로의 전환을 선택했습니다.

### 2. 멀티모듈(Multi-module) 구조로의 전환과 기대효과
  모놀리틱 구조로부터 발생한 문제를 극복하기 위해, 각 기능이 모듈별로 독립적으로 관리되고 확장성이 높은 멀티모듈 구조로의 전환을 시도했습니다. 시스템을 여러 개의 모듈로 분리하여, 각 모듈이 명확한 책임과 의존성을 가지도록 합니다. 멀티 모듈 구조로 전환함에 따라 기대할 수 있었던 장점은 아래와 같습니다.

1. 코드 중복 감소 및 도메인 분리
   - 모듈 단위로 시스템을 분리 함에 따라 코드 중복을 최소화할 수 있습니다. 각 모듈은 주어진 책임에 집중할 수 있고, 여러 모듈에서 공통적으로 필요한 코드는 별도의 모듈에서 관리하는 등의 방법을 선택할 수 있습니다. 이를 통해 코드의 재사용성을 크게 높이고 유지보수성을 높일 수 있을 것입니다.
2. 관리 포인트의 감소
   - 각 모듈의 책임과 의존성이 명확히 정의되어있다면, 개발자는 작업의 영향 범위를 쉽게 파악할 수 있어 관리 포인트가 줄어듭니다. 특정 기능을 수정하거나 변경할 때, 어느 모듈에 영향이 있을 지 쉽게 파악하며 과감한 작업을 시도할 수 있습니다.
3. 확장성, 유연성 개선
   - 특정 기능에 대한 확장이 필요한 경우에, 해당 모듈에 대해서만 작업하면 되기 때문에 시스템의 안정성을 걱정하지 않고 작업이 가능해집니다.
4. MSA 로의 전환 가능성
   - 멀티모듈 구조는 MSA 로의 전환을 위한 중간 과정의 역할을 할 수 있습니다. 트래픽에 따라 각 모듈을 필요 시 독립적인 서비스로 확장하기에 유리합니다. 만약 모놀리틱 구조에서 한번에 MSA 로의 전환이 필요하다면, 굉장히 큰 공수가 들 것으로 예상합니다만, 모듈이 잘 분리된 멀티 모듈 구조의 프로젝트라면 훨씬 적은 리스크로 전환할 수 있을 것 입니다. 

이 외에도 각 팀원이 작업할 때, 요구사항을 보고 어떤 모듈에서 작업해야 할 지 작업 계획/전략을 수립하기 쉬워지며, 새로운 팀원이 합류 했을 때 프로젝트에 더 쉽게 적응할 수 있을 것으로 생각됩니다! 아무쪼록, 멀티 모듈로의 전환은 대피로 백엔드에 있어서 아주 긍정적인 효과가 있을 것으로 예상됩니다 ㅎㅎ

### 3. 모듈 설계를 위한 고민

  그렇다면 모듈을 어떻게 분리하는 게 좋을까요? 이 부분이 가장 어려웠던 것 같네요.. 대피로 백엔드에서는 각 모듈의 `책임` 과 `의존성` 을 중심으로 모듈을 분리하는 것으로 컨셉을 설정했습니다. 멀티 모듈 구조로 전환함에 따라 긍정적인 효과를 얻기 위해서는 결국 각 모듈 간의 책임이 명확해야 하고, 합리적으로 의존 관계가 정의되는 것이 핵심이라고 생각했기 때문이었죠. 각 모듈은 하나의 책임을 가지도록 하고(SRP), 모듈 간 의존성은 단방향으로 흐르고 순환 참조가 일어나지 않아야 합니다. 

우리가 설계한 구조는 아래와 같습니다.

<img width="852" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/d0eaa0b1-4d53-4f35-adaa-0f065f72de00">


- **daepiro-core**
  - 어플리케이션의 데이터 스토리지와 관련된 모든 책임을 가집니다. 데이터베이스 설정, 엔티티 관리, 쿼리 단 코드들이 포함됩니다.
- **daepiro-redis**
  - 레디스 설정과 구현 부분을 독립적으로 담당합니다. 
- **daepiro-api**
  - 비지니스 로직에 대한 책임을 담당합니다. 사용자 요청을 처리하고 적절한 서비스 로직을 수행하여 응답을 반환하도록 합니다.
- **daepiro-auth**
  - 인증, 인가 메커니즘을 관리합니다.
- **daepiro-common**
  - 어플리케이션 전반에서 공통적으로 사용될 수 있는 유틸리티 기능을 담당합니다. 단, common module 이 너무 무거워지지 않도록 경계해야 합니다. common module 의 변경사항은, 일반적으로 전체 module 에 영향을 미칠 수 있기 때문입니다.
- **daepiro-crawler**
  - 대피로 프로젝트는 일정 주기마다 재난 알림 사이트를 크롤링하여 데이터를 수집합니다. 크롤러 모듈에서는 크롤링을 통한 데이터 수집 로직만을 담당하도록 합니다.
- **daepiro-app**
  - 엔트리 모듈으로, 어플리케이션의 진입점을 제공하는 역할을 담당합니다.

각 모듈의 경계를 명확하게 분리하고, 의존성을 최소화 하면서 반드시 필요한 경우에만 설정할 수 있도록 고민했는데요! 개발을 계속 진행하면서, 현재 구조가 정말 괜찮은 지 팀원들과 지속적으로 검토하며 다양한 시도를 해보려고 합니다. 최적의 구조는 어떤 모습일까요?  

### 4. 멀티모듈로의 전환

Grade 을 사용하여 멀티모듈 구조를 쉽고 효과적으로 관리할 수 있습니다.

<img width="1358" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/bec3ed51-13bf-4542-a01b-2d3811d6a859">



 `settings.gradle` 에서는 아래와 같이 모듈 설정을 해주어야 합니다.

```gradle
rootProject.name = 'Backend'
include 'daepiro-app'
include 'daepiro-api'
include 'daepiro-core'
include 'daepiro-common'
include 'daepiro-redis'
include 'daepiro-auth'
include 'daepiro-crawler'
```

루트 프로젝트의 `build.gradle` 설정입니다.
- 여기서는 전체 프로젝트에 대한 공통 설정을 제공합니다.

```gradle
plugins {
    // 필요한 plugin 설정을 root 에서 해둡니다.
    id 'java'
    // spring boot 버전을 관리합니다.
    id 'org.springframework.boot' version '3.1.4'
    id 'io.spring.dependency-management' version '1.1.3'
    // 대피로 백엔드에서는 jib 를 이용하여 도커 이미지를 빌드하고 있기 때문에 설정했습니다.
    id 'com.google.cloud.tools.jib' version '3.4.0'
}

group = 'com.numberone.backend'
version = '0.0.2-SNAPSHOT'

allprojects {
    apply plugin: 'java'
    // 자바 버전을 변경하고자 한다면, 아래 설정을 변경하면 됩니다.
    sourceCompatibility = JavaVersion.VERSION_17

    repositories {
        mavenCentral()
        maven { url 'https://jitpack.io' }
    }
}

ext {
    set('springCloudVersion', "2021.0.1")
}

subprojects {
    apply plugin: 'io.spring.dependency-management'
    apply plugin: 'java-library'
    apply plugin: 'org.springframework.boot'
    apply plugin: 'jacoco'

    configurations {
        compileOnly {
            extendsFrom annotationProcessor
        }
    }

    repositories {
        mavenCentral()
    }

    dependencyManagement {
        imports {
            mavenBom("org.springframework.boot:spring-boot-dependencies:3.1.4")
            mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
        }
    }

    dependencies {
        // lombok
        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
        annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'

        // spring
        implementation 'org.springframework.boot:spring-boot-starter-web'
        implementation 'org.springframework.boot:spring-boot-starter-validation'
        implementation 'org.springframework.boot:spring-boot-starter-security'
        implementation 'org.springframework.boot:spring-boot-starter-aop'

        // swagger
        implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.0.2'

    }

}
```


- core module

```gradle
dependencies {
 implementation project(':daepiro-common')

 // p6spy
 implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.8'

 // jpa
 api 'org.springframework.boot:spring-boot-starter-data-jpa'

 // mysql connector
 runtimeOnly 'com.mysql:mysql-connector-j'

 // === QueryDsl  ===
 implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
 annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"
 annotationProcessor "jakarta.annotation:jakarta.annotation-api"
 annotationProcessor "jakarta.persistence:jakarta.persistence-api"
 // === QueryDsl  ===
}

// === Querydsl 빌드 옵션  ===
def querydslDir = "$buildDir/generated/querydsl"

sourceSets {
 main.java.srcDirs += [ querydslDir ]
}

tasks.withType(JavaCompile) {
 options.annotationProcessorGeneratedSourcesDirectory = file(querydslDir)
}

clean.doLast {
 file(querydslDir).deleteDir()
}
// === Querydsl 빌드 옵션  ===

bootJar {
 enabled = false
}
jar {
 enabled = true
}

```

- redis module

```gradle
dependencies {
    implementation project(':daepiro-common')

    // redis
    api 'org.springframework.boot:spring-boot-starter-data-redis'
}

bootJar {
    enabled = false
}
jar {
    enabled = true
}

```

- common module


```gradle
dependencies {
    // fcm
    implementation 'com.google.firebase:firebase-admin:9.1.1'

    //geocoding
    implementation 'com.google.code.geocoder-java:geocoder-java:0.16'
}

bootJar {
    enabled = false
}
jar {
    enabled = true
}

```

- auth module

```gradle
dependencies {
    implementation project(':daepiro-core')
    implementation project(':daepiro-redis')
    implementation project(':daepiro-common')

    // oauth
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-resource-server'

    // jwt
    implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
    runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
    runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'

}

bootJar {
    enabled = false
}
jar {
    enabled = true
}

```

- api module

```gradle
dependencies {
 implementation project(':daepiro-core')
 implementation project(':daepiro-common')
 implementation project(':daepiro-redis')
}

bootJar {
 enabled = false
}

jar {
 enabled = true
}

```

- crawler module

```gradle
dependencies {
    implementation project(':daepiro-common')
    implementation project(':daepiro-core')

    //crawling
    implementation 'com.squareup.okhttp3:okhttp:4.9.1'
    implementation 'org.jsoup:jsoup:1.14.2'
    implementation 'net.sourceforge.htmlunit:htmlunit:2.70.0'
}

bootJar {
    enabled = false
}
jar {
    enabled = true
}

```

멀티모듈 프로젝트에서는 각 모듈의 책임에 따라 실행 가능한 스프링부트 어플리케이션으로 패키징 될 필요가 있는지 고려해야 하는데요, gradle 에서는 bootJar, jar task 에 대한 enabled flag 를 설정하여 제어할 수 있습니다.

스프링 부트 어플리케이션으로 실행될 필요가 없는 모듈의 경우에는 아래와 같이 bootJar task 를 비활성화하고 jar 로의 패키징만 수행되어 다른 모듈에서 필요 시 포함할 수 있도록 설정할 수 있습니다. 

```gradle
bootJar {
    enabled = false
}
jar {
    enabled = true
}
```

엔트리 모듈에서는 bootJar 를 true 로 설정하여, 실행가능한 스프링 부트 어플리케이션으로 동작할 수 있도록 해야겠죠. 
  추후에 batch 모듈을 추가할 계획이 있는데요, batch application 을 독립적으로 실행될 수 있도록 구성하기 위해 batch 모듈의 gradle 설정 부분에서는 bootJar 부분을 enable 해주어야 할 것 같습니다.

- app module

```gradle
plugins {
    id 'com.google.cloud.tools.jib' version '3.4.0'
}

dependencies {
    implementation project(':daepiro-core')
    implementation project(':daepiro-common')
    implementation project(':daepiro-redis')
    implementation project(':daepiro-api')
    implementation project(':daepiro-crawler')
    implementation project(':daepiro-auth')
}

// system environment variables
def serverIP = System.getenv("EC2_PUBLIC_IP")
def jmxPort = System.getenv("JMX_PORT")
def dockerUserName = System.getenv("DOCKER_USERNAME")
def dockerImageName = System.getenv("DOCKER_IMAGE")

// jib
jib {
    from {
        image = "openjdk:17"
    }
    to {
        image = String.format("%s/%s", dockerUserName, dockerImageName)
        tags = ["latest"]
    }
    container {
        mainClass = 'com.numberone.backend.BackendApplication'
        creationTime = "USE_CURRENT_TIMESTAMP"
        jvmFlags = [
                "-Duser.timezone=Asia/Seoul",
                "-Xms128m", "-Xmx128m",
                "-Dcom.sun.management.jmxremote=true",
                "-Dcom.sun.management.jmxremote.local.only=false",
                "-Dcom.sun.management.jmxremote.port=" + jmxPort.toString(),
                "-Dcom.sun.management.jmxremote.ssl=false",
                "-Dcom.sun.management.jmxremote.authenticate=false",
                "-Djava.rmi.server.hostname=" + serverIP.toString(),
                "-Dcom.sun.management.jmxremote.rmi.port=" + jmxPort.toString(),
                "-XX:+HeapDumpOnOutOfMemoryError",
                "-XX:HeapDumpPath=/heap-dumps/heapdump.hprof"]
        ports = ['8080']
    }
}

bootJar {
    enabled = true 
}
jar {
    enabled = true
}

```

- app module 은 entry module 의 역할을 수행합니다.
- 멀티모듈 프로젝트에서 jib 기반으로 이미지를 빌드하기 위해서는 mainClass 를 명시하지 않으면, jib 에서 메인 클래스를 찾지 못하여 실패합니다.
  - 아래와 같이 mainClass path 를 명시해주도록 합시다.
    - `mainClass = 'com.numberone.backend.BackendApplication'`
    - 외에 jvmFlags 는 필요에 따라 추가하고 제거해주도록 합시다.


세부 코드는 아래 레포지토리에서 확인할 수 있습니다.
- https://github.com/Team-NumberOne/Backend

### 5. 멀티모듈로 전환하며 생긴 고민들

우리 프로젝트에서 위 구조가 과연 최적일까요..?
각 모듈 간 책임과 의존관계가 합리적일까요..?

팀원과 함께 많이 고민하며 설계한 구조이지만, 현재의 구조가 항상 최적이 아닐 수 있다는 점을 명확히 인지하며 작업하려고 합니다. 앞으로 발생할 추가적인 요구사항들을 개발하거나, 리팩토링하면서 각각의 모듈의 책임을 몸소 경험하며 최종적으로는 더 나은 구조로 나아갈 수 있도록 해야겠죠!

추가적으로 각 모듈에서 필요한 프로퍼티 파일들을 어떻게 관리할 지 고민이 있습니다.
현재는 하나의 application.yml 에 모든 설정을 몰아두어 관리하고 있는데요, 여러 모듈의 설정 파일이 하나에 몰려있다는 것은 결코 좋은 방식이 아닐 것입니다. 결국, 모듈 별로 독립된 application.yml 파일을 생성해두면 될 텐데요, 현재 배포 방식에서 application.yml 은 github secrets 으로 관리하고 있습니다.

그리고 아래와 같이 directory 에 직접 파일을 생성해서 사용하고 있죠.

```
- name: 🐧 create application.yml
  run: |
    mkdir -p ./daepiro-app/src/main/resources
    cd ./daepiro-app/src/main/resources
    touch ./application.yml
    echo "${{ secrets.PROPERTIES_PROD }}" | base64 --decode > ./application.yml
    ls -la
  shell: bash
```

각 모듈 별로 application.yml 이 분리된다면,
각 모듈 별 PROPERTY 를 GitHub secrets 에 별도로 저장해두어야 할 것 입니다.
개발하며 변경사항이 생기면, 그때마다 수동으로 github secrets 을 변경한 뒤
팀원에게 수정사항을 설명해야 하는 과정이 필연적일텐데요.. 생산성이 많이 떨어질 것 같습니다.
더불어,  GitHub secrets 은 변경 history 가 남지 않기 때문에 잘못 변경했을때 서비스 장애로 이어지는 큰 리스크가 존재하죠. 사람의 실수가 서비스 장애로 온전히 이루어지는 이 구조는 정말 좋지 않은 것 같습니다. 어떻게 해결할 수 있을까요? 여러 방면으로 고민 중인데요, 얼른 해결해서 포스팅하고싶네요!

### 6. 기존의 배포 방식의 문제점, 그리고 blue - green 배포로의 전환
  기존의 대피로 백엔드 배포는 동작하던 컨테이너를 내리고, 이미지를 다시 받아와서 컨테이너로 띄우기 때문에 1분 가량의 다운타임이 발생하는 치명적인 문제가 있었습니다. 앞으로 실 서비스가 된다면, 반드시 문제가 될 요소였기 때문에, 이번 작업에 포함하여 무중단 배포가 일어나도록 개선하였습니다.

  선택한 방법은 blue-green 배포인데요, 현재 서비스 중인 환경(blue) 외에 새로운 버전을 배포할 환경(green)을 모두 구성한 뒤, nginx 를 이용하여 새로운 버전이 완전히 배포되고 health-check 가 된다면 트래픽을 blue 에서 green 으로 한번에 전환하는 방식입니다.

![image](https://github.com/versatile0010/versatile0010.github.io/assets/96612168/278fdc44-8be0-4478-ae9e-afc8050830e1)


  이를 통해 사용자는 다운타임을 거의 체감하지 못하고(nginx 를 reload 하는 정도의 다운타임이 존재하지만 매우 짧기 때문에 일반적으로 인지하기 어려울 것) 서비스를 지속적으로 이용할 수 있습니다. 
  또한 롤백이 필요할 경우에 이전 버전의 컨테이너로 곧바로 트래픽을 전환하면서 빠르게 롤백할 수 있죠.

( 여담 )
   가난한.. 대피로 백엔드는 aws ec2 t2.micro 환경에서 돌아가고 있는데요,, 대피로 스프링 어플리케이션 컨테이너를 동시에 2개 이상 띄우는 것은 주어진 서버 리소스 상 어려울 것으로 판단하였습니다. 아직 수익이 발생하지 않아 서버 사양을 높일 수가 없어, t2.micro 를 사용하고 있는 현재, 임시적으로 하나의 스프링 컨테이너만 살아있도록 조치해두고 있습니다. 가령, blue 에 새로운 버전을 배포하게 된 경우라면 blue 를 새로 띄우고 green 은 롤백 시 사용하기 위해 살려두어야 하지만, 지금은 green 을 kill 하고 있죠..


  적용은 매우 간단합니다. 대피로 백엔드에서는 (자금 절약 이슈로) blue, green 컨테이너를 한 서버에서 띄워야 했으므로 port 를 다르게 하여 두 환경을 구분했습니다.

  docker-compose 에서 blue, green 에 대한 ext - in port 를 각각 8081:8080, 8082:8080 으로 설정해두었어요. 

```docker
version: '3'
services:
  blue:
    image: your-docker-image
    container_name: blue
    restart: always
    ports:
      - 8081:8080

  green:
    image: your-docker-image
    container_name: green
    restart: always
    ports:
      - 8082:8080
...
```

  또한 blue, green 용도로 사용할 nginx.conf 를 아래와 같이 설정해두었습니다.
- blue
  
```nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name localhost;

        location / {
                proxy_pass http://localhost:8081;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }

        # pass PHP scripts to FastCGI server
        #
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
                fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}

```
 
- green

```nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name localhost;

        location / {
                proxy_pass http://localhost:8082;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }

        # pass PHP scripts to FastCGI server
        #
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
                fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}
```

배포 쉘 스크립트는 아래와 같아요.

```shell
#!/bin/bash

IS_GREEN_EXIST=$(docker ps | grep green)
DEFAULT_CONF=" /etc/nginx/nginx.conf"

# blue가 실행 중이면 green을 up합니다.
if [ -z $IS_GREEN_EXIST ];then
  echo "### BLUE => GREEN ####"
  echo ">>> green image를 pull합니다."
  docker-compose pull green
  echo ">>> green container를 up합니다."
  docker-compose up -d green
  while [ 1 = 1 ]; do
  echo ">>> green health check 중..."
  sleep 3
  REQUEST=$(curl http://127.0.0.1:8082/hello)
    if [ -n "$REQUEST" ]; then
      echo ">>> 🍃 health check success !"
      break;
    fi
  done;
  sleep 3
  echo ">>> nginx를 다시 실행 합니다."
  sudo ln -s -f /etc/nginx/sites-available/green /etc/nginx/sites-enabled/default
  sudo nginx -s reload
  echo ">>> blue container를 down합니다."
  docker-compose stop blue

# green이 실행 중이면 blue를 up합니다.
else
  echo "### GREEN => BLUE ###"
  echo ">>> blue image를 pull합니다."
  docker-compose pull blue
  echo ">>> blue container up합��다."
  docker-compose up -d blue
  while [ 1 = 1 ]; do
    echo ">>> blue health check 중..."
    sleep 3
    REQUEST=$(curl http://127.0.0.1:8081/hello)
    if [ -n "$REQUEST" ]; then
      echo ">>> 🍃 health check success !"
      break;
    fi
  done;
  sleep 3
  echo ">>> nginx를 다시 실행 합니다."
  sudo ln -s -f /etc/nginx/sites-available/blue /etc/nginx/sites-enabled/default
  sudo nginx -s reload
  echo ">>> green container를 down합니다."
  docker-compose stop green
fi
```

### 7. 앞으로의 과제는?
  이번 포스팅에서는 사이드 프로젝트인 대피로의 아키텍쳐를 멀티모듈 구조로 전환한 것과, blue-green 무중단 배포를 적용한 경험에 대해서 작성해보았습니다! 이제 대피로 백엔드를 조금 더 발전시키기 위해 다른 과제를 해보려고 해요! 지금 살펴보고 있는 것은.. 현재의 구조에서 테스트가 용이하도록 하려면 어떻게 해야할 지에 대한 내용입니다.
  지금 대피로 백엔드에 존재하는 가장 큰 문제는 일부 api 들이 mysql 에 완전히 의존적인 부분이 있다는 것인데요, 일반적으로 테스트 환경에서는 h2 database 를 이용하는 경우가 많은데 현재 대피로 백엔드에서는 불가능하죠. 이 부분을 어떻게 해결할 수 있을까요? 사실 프로덕션 코드가 특정 db 구현체에 의존하고 있는 상황은 좋지 않다고 생각합니다. mysql 이라는 구현체에 의존하고 있는 프로덕션 코드를 리팩토링하여, 테스트가 용이하도록 해볼 계획이에요!! 해당 작업이 완료되면 조만간 다시 포스팅하도록 하겠습니다! 감사합니다 :)

---

(24.03.07 추가)
: 일반적인 테스트 환경에서 h2 를 이용하는 이유는 편리성 때문일텐데요, 프로덕션 코드의 핵심 로직을 테스트 할 수 없음에도 h2 를 고집해야 할까요!? 이 부분은 [🚃 더 빠르게 달리기 위해 잠깐 멈추기](https://versatile0010.github.io/test/sideproject/test-container-java/) 포스팅에서 다뤄보도록 하겠습니다!


### References
1. [What is Blue Green Deployment by Shashank Srivastava Apr 9, 2021](https://www.opsmx.com/blog/blue-green-deployment/)
2. [Blue Green Deployment](https://medium.com/practo-engineering/blue-green-deployment-on-amazon-aws-38b820518411)
3. [Creating a Gradle multi-module project](https://tmsvr.com/gradle-multi-module-build/)
