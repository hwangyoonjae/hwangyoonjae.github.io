---
layout: post
title: "Jenkins Pipeline 구축하기"
date: 2023-10-19
categories: [DevOps, Jenkins]
tags: [Jenkins, CI, CD, Github]
image: /assets/img/post-title/jenkins-wallpaper.jpg
---

## 1. Jenkins Pipeline 구축하기 :
### 1.1 Pipeline 아이템 생성하기 :
- Jenkins 접속하여 아이템 생성 클릭 후, ***Pipeline***를 선택합니다.

![Pipeline 아이템 생성하기](/assets/img/post/Jenkins/Pipeline%20아이템%20생성하기.png)

* * *

### 1.2 Pipeline 깃연결하기 :
- 아이템의 설명과 깃을 연결합니다.

![Pipeline 설명과 깃연결](/assets/img/post/Jenkins/Pipeline%20설명과%20깃연결.png)

* * *

### 1.3 Pipeline script 작성하기 :

- 파이프라인이 어떤 동작을 할 지 스크립트로 내용을 작성합니다.

![Pipeline 스크립트](/assets/img/post/Jenkins/Pipeline%20스크립트.png)

* * *

### 1.4 스크립트 내용 상세보기 :
- 위에서 작성한 파이프라인 스크립트는 아래와 같습니다.

```bash
pipeline {
    agent any
    
    stages {
        stage('git clone') {
            steps {
                script {
                    // Git clone 단계 내에서 필요한 작업을 수행
                    checkout([$class: 'GitSCM', branches: [[name: 'main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: '깃서버주소']]])
                }
            }
        }
        
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
  
        stage('execute sh') {
            steps {
                script {
                  // 빌드 스크립트를 파일에 저장
                  writeFile file: 'project.sh', text: '#!/bin/bash\n\n' + 
                      # 여기에 프로젝트 빌드 스크립트 또는 작업을 추가\n' + 
                      '\n' + 
                      '# 프로젝트 빌드나 작업 성공 여부 확인\n' + 
                      'if [ $? -eq 0 ]; then\n' + 
                      '    echo "성공했습니다"\n' + 
                      'else\n' + 
                      '    echo "실패했습니다"\n' + 
                      '    exit 1\n' + 
                      fi\n'
              }
              sh "chmod 774 ./project.sh"
              sh "./project.sh"
            }
        }        
    }
}
```

* * *

### 1.6 Pipeline 빌드하기 :
- 위에서 만든 스크립트를 통해 Pipeline을 빌드합니다.

![Pipeline 빌드 실행과 결과](/assets/img/post/Jenkins/Pipeline%20빌드%20실행과%20결과.png)

* * *