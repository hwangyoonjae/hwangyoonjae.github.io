---
layout: post
title: "CICD 파이프라인을 통해 Nexus 라이브러리 등록하기"
date: 2025-09-25
categories: [DevOps, Gitlab, Nexus]
tags: [Git, Project, Runner]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## Junit 테스트 소스 :
### Maven(JUnit) 테스트용 잡 :

```bash
maven-test:
  stage: test
  image: harbor.test.com/junit/maven:3.9.6-eclipse-temurin-17
  # extends: [.default_prep, .trust_ca]
  before_script:
    - !reference [.default_prep, before_script]
    - !reference [.trust_ca, before_script]
    # Maven JVM에 truststore 적용
    - export MAVEN_OPTS="$MAVEN_OPTS -Djavax.net.ssl.trustStore=$CI_PROJECT_DIR/cacerts -Djavax.net.ssl.trustStorePassword=changeit"
  script:
    - cd maven-app
    - mvn -B -s "$CI_PROJECT_DIR/ci/maven/settings.xml" test
  artifacts:
    when: always
    reports:
      junit: "$CI_PROJECT_DIR/maven-app/target/surefire-reports/TEST-*.xml"
    paths:
      - "$CI_PROJECT_DIR/maven-app/target/surefire-reports/"
```

* * *

### Gradle(JUnit) 테스트용 잡 :

```bash
gradle-test:
  stage: test
  image: harbor.test.com/junit/gradle:8.10.0-jdk17
  # extends: [.default_prep, .trust_ca]
  before_script:
    - !reference [.default_prep, before_script]
    - !reference [.trust_ca, before_script]
    # Gradle JVM(모든 워커)에 truststore 적용
    - export JAVA_TOOL_OPTIONS="-Djavax.net.ssl.trustStore=$CI_PROJECT_DIR/cacerts -Djavax.net.ssl.trustStorePassword=changeit $JAVA_TOOL_OPTIONS"
  script:
    - cd gradle-app
    - if [ -f ./gradlew ]; then chmod +x ./gradlew && ./gradlew --no-daemon -I "$CI_PROJECT_DIR/ci/gradle/init.gradle" test; else gradle --no-daemon -I "$CI_PROJECT_DIR/ci/gradle/init.gradle" test; fi
  artifacts:
    when: always
    reports:
      junit: "$CI_PROJECT_DIR/gradle-app/build/test-results/test/TEST-*.xml"
    paths:
      - "$CI_PROJECT_DIR/gradle-app/build/test-results/test/"
  needs: [maven-test]
```

* * *

## Nexus Mirror 등록하기 :
### Maven Mirror 등록하기 :

```bash
# setting.xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                              https://maven.apache.org/xsd/settings-1.0.0.xsd">

  <mirrors>
    <mirror>
      <id>nexus-group</id>
      <url>${env.NEXUS_GROUP_URL}</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
  </mirrors>

  <servers>
    <server>
      <id>nexus-group</id>
      <username>${env.NEXUS_USER}</username>
      <password>${env.NEXUS_PASSWORD}</password>
    </server>
  </servers>

  <profiles>
    <profile>
      <id>use-nexus-group</id>
      <repositories>
        <repository>
          <id>nexus-group</id>
          <url>${env.NEXUS_GROUP_URL}</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
    </profile>
  </profiles>

  <activeProfiles>
    <activeProfile>use-nexus-group</activeProfile>
  </activeProfiles>
</settings>
```

* * *

### Gradle Mirror 등록하기 :

```bash
# init.gradle
allprojects {
    buildscript {
        repositories {
            all { remove(this) }
            maven {
                url System.getenv("NEXUS_GROUP_URL")
                credentials {
                    username System.getenv("NEXUS_USER")
                    password System.getenv("NEXUS_PASSWORD")
                }
            }
        }
    }

    repositories {
        all { remove(this) }
        maven {
            url System.getenv("NEXUS_GROUP_URL")
            credentials {
                username System.getenv("NEXUS_USER")
                password System.getenv("NEXUS_PASSWORD")
            }
        }
    }
}
```

* * *

## JAVA truststore의 인증서(Corporate CA) 추가하기 :
### CI/CD 파이프라인의 잡 추가하기:
```bash
.trust_ca:
  before_script:
    - echo "$CORP_CA_PEM_B64"
    - echo "$CORP_CA_PEM_B64" | base64 -d > "$CI_PROJECT_DIR/corp-ca.pem"
    - |
      CACERTS_SRC="${JAVA_HOME}/lib/security/cacerts"
      [ -f "$CACERTS_SRC" ] || CACERTS_SRC="$(readlink -f /etc/ssl/certs/java/cacerts || true)"
      echo "Using cacerts: $CACERTS_SRC"
      cp "$CACERTS_SRC" "$CI_PROJECT_DIR/cacerts"
    - |
      ALIAS=corp-ca
      STORE="$CI_PROJECT_DIR/cacerts"
      PASS=changeit
      CERT="$CI_PROJECT_DIR/corp-ca.pem"

      if keytool -list -keystore "$STORE" -storepass "$PASS" -alias "$ALIAS" >/dev/null 2>&1; then
        echo "Alias $ALIAS already exists, skipping import."
      else
        echo "Importing certificate $CERT as alias $ALIAS ..."
        keytool -importcert -noprompt -trustcacerts \
          -alias "$ALIAS" \
          -file "$CERT" \
          -keystore "$STORE" \
          -storepass "$PASS"
      fi
    - keytool -list -keystore "$CI_PROJECT_DIR/cacerts" -storepass changeit -alias corp-ca || true
```

* * *

### GitLab CI 변수에 Base64로 인코딩된 사내 CA 인증서 저장하기 :

- 사내 CA 인증서를 Base64로 인코딩하여 결과 값을 복사한다.

```bash
$ base64 -w0 ca.crt > ca.crt.b64

# base64 인코딩된 인증서 내용 확인하여 복사
$ cat ca.crt.b64
```

![사내 CA 인증서 base64 인코딩](/assets/img/post/Gitlab/사내%20CA%20인증서%20base64%20인코딩.png)

* * *

- 복사한 인코딩 값을 Gitlab CI 변수에 저장한다.

![인증서 인코딩 값 Gitlab 변수 저장](/assets/img/post/Gitlab/인증서%20인코딩%20값%20Gitlab%20변수%20저장.png)

> CI 파이프라인을 통해 빌드된 라이브러리를 Nexus Hosted 저장소에 배포(업로드)한다.
{: .prompt-info}

* * *