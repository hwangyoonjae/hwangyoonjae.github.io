---
layout: post
title: "CICD 프로젝트 Runner 자동 등록 파이프라인"
date: 2025-08-20
categories: [DevOps, Gitlab]
tags: [Git, Project, Runner]
image: /assets/img/post-title/gitLab-wallpaper.jpg
---

## 프로젝트 Runner 자동 등록 파이프라인 생성하기:

> 필자는 프로젝트에서 사용자들의 요청으로 해당 파이프라인을 생성하였습니다.
{: .prompt-info}

```yml
# Gitlab 파이프라인 단계 정의
stages:
  - register-runner

.add-register-runner:
  script:
    - |
      echo "$(whoami)"
      set -x
      GITLAB_TOKEN="[Access_Token 값]"
      GITLAB_API_URL="https://[Gitlab_URL]:[Port]"

      # 프로젝트 ID와 이름을 저장할 배열을 선언
      declare -a PROJECT_IDS
      declare -a PROJECT_NAMES
      # API 페이지네이션을 위한 변수 설정
      page=1

      # GitLab 그룹(ID 74)에 속한 모든 프로젝트를 페이지별로 조회하는 반복문
      while :; do
        response=$(curl --insecure --header "PRIVATE-TOKEN: ${GITLAB_TOKEN}" "${GITLAB_API_URL}/api/v4/groups/[Group_ID]/projects?include_subgroups=true&per_page=20&page=${page}")
        
        # jq를 사용하여 API 응답(JSON)에서 프로젝트 ID와 이름을 추출
        projects_dates=$(echo "$response" | jq -r '.[] | "\(.id) \(.name)"')

        # 더 이상 프로젝트가 없으면(응답이 비어있으면) 반복문을 종료
        if [ -z "$projects_dates" ]; then
          break
        fi

        # 추출된 프로젝트 ID와 이름을 줄 단위로 읽어 배열에 추가
        while IFS= read -r line; do
          project_id=$(echo "$line" | awk '{print $1}')
          project_name=$(echo "$line" | awk '{$1=""; print $0}')
          PROJECT_IDS+=("$project_id")
          PROJECT_NAMES+=("$project_name")
        done <<< "$projects_dates"

        ((page++))
      done

      echo "gitlab project runner"
      # 모든 프로젝트를 순회하며 Runner를 등록하는 반복문
      for i in "${!PROJECT_IDS[@]}"; do
        project_id=$(echo "${PROJECT_IDS[$i]}" | awk '{$1=$1};1')
        project_name=$(echo "${PROJECT_NAMES[$i]}" | awk '{$1=$1};1')
        echo "Fetching Runner Token for Project: ${project_name}"

        # 해당 프로젝트에 현재 등록된 Runner가 있는지 확인
        RUNNER_COUNT=$(curl --silent --header "PRIVATE-TOKEN: ${GITLAB_TOKEN}" "${GITLAB_API_URL}/api/v4/projects/${project_id}/runners" | jq -r 'length')
        
        # Runner가 0개(미존재)일 경우에만 등록을 진행
        if [ "$RUNNER_COUNT" -eq 0 ]; then
          echo "${project_name}, ID:${project_id} - Runner 미존재"

          # Runner 등록을 위한 토큰 조회
          RUNNER_TOKEN=$(curl --insecure --header "PRIVATE-TOKEN: ${GITLAB_TOKEN}" "${GITLAB_API_URL}/api/v4/projects/${project_id}" | jq -r '.runners_token')

          # 토큰을 성공적으로 가져왔는지 확인
          if [ -z "$RUNNER_TOKEN" ]; then
            echo "Failed to fetch Runner token"
          else
            echo "$project_name Runner Token:$RUNNER_TOKEN"
            gitlab-runner register --non-interactive --url "$GITLAB_API_URL" --registration-token "$RUNNER_TOKEN" --executor "shell" --description "$project_name" --locked="false" --run-untagged="true" --tag-list "${project_name}" 
          fi
        fi
      done

      # gitlab-runner restart

# 실제로 실행되는 Job
copy_project:
  stage: register-runner
  # .add-register-runner 템플릿의 스크립트를 상속받아 실행
  extends: .add-register-runner
  tags:
    - cicd-build
  rules:
    - if: '$CI_COMMIT_BRANCH == null && $CI_COMMIT_TAG =~ /.*add_runner.*/'
```