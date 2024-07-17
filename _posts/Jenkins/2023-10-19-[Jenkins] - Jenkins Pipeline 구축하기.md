---
layout: post
title: "[Jenkins] - Jenkins Pipeline 구축하기"
date: 2023-10-19
categories: Jenkins
tags: [Jenkins, CI, CD, Github]
image: /assets/post/jenkins-wallpaper.jpg
---

## Jenkins Pipeline 구축하기:
### Pipeline 아이템 생성하기:
- Jenkins 접속하여 아이템 생성 클릭 후, ***Pipeline***를 선택한다.
[![Pipeline 아이템 생성하기](/assets/images/Jenkins/Pipeline%20아이템%20생성하기.png)](/assets/images/Jenkins/Pipeline%20아이템%20생성하기.png)

* * *

### Pipeline 깃연결하기:
- 아이템의 설명과 깃을 연결한다.
[![Pipeline 설명과 깃연결](/assets/images/Jenkins/Pipeline%20설명과%20깃연결.png)](/assets/images/Jenkins/Pipeline%20설명과%20깃연결.png)

* * *

### Pipeline script 작성하기:

- 파이프라인이 어떤 동작을 할 지 스크립트로 내용을 작성한다.
[![Pipeline 스크립트](/assets/images/Jenkins/Pipeline%20스크립트.png)](/assets/images/Jenkins/Pipeline%20스크립트.png)

* * *

### 스크립트 내용 상세보기:
- 위에서 작성한 파이프라인 스크립트는 아래와 같다.
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

### Pipeline 빌드하기:
- 위에서 만든 스크립트를 통해 Pipeline을 빌드한다.
[![Pipeline 빌드 실행과 결과](/assets/images/Jenkins/Pipeline%20빌드%20실행과%20결과.png)](/assets/images/Jenkins/Pipeline%20빌드%20실행과%20결과.png)

* * *