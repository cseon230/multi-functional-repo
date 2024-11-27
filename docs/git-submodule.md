# 서브모듈 추가
git submodule add https://github.com/junsun0708/jyh-projectB-git-submodule.git repos/jyh/jyh-projectB-git-submodule  
git submodule add https://github.com/junsun0708/jyh-projectA-git-submodule.git repos/jyh/jyh-projectA-git-submodule

# 서브모듈 초기화 및 업데이트
git submodule init
git submodule update

# 모든 서브모듈 업데이트 (각각의 최신 커밋 가져오기)
git submodule update --remote
  
  
---
# **아래의 내용은 CursorAI를 사용하여 작성**
--- 

# Git 서브모듈(Submodule)이란?
Git 서브모듈은 Git 저장소 안에 다른 Git 저장소를 디렉토리로 분리해서 넣는 방법입니다. 
이를 통해 다른 독립적인 Git 저장소를 자신의 Git 저장소 안에서 특정 버전으로 관리할 수 있습니다.

## 서브모듈의 장점
- 외부 저장소의 코드를 프로젝트에 포함하면서 해당 코드의 이력을 별도로 관리할 수 있음
- 특정 버전으로 고정해서 사용 가능
- 여러 프로젝트에서 공통으로 사용하는 코드를 효율적으로 관리 가능

## 서브모듈의 단점
- 저장소 관리가 복잡해질 수 있음
- 서브모듈 업데이트 시 수동으로 해야 함
- 초기 클론 시 추가 명령어 필요

# Git 서브트리(Subtree)란?
서브트리는 서브모듈의 대안으로 사용되는 방식으로, 하나의 저장소 안에 다른 저장소의 콘텐츠를 포함시키는 방법입니다.

## 서브트리의 장점
- 서브모듈보다 사용이 간단함
- 기존 저장소의 히스토리가 모두 포함됨
- 별도의 초기화나 업데이트 명령이 필요 없음

## 서브트리의 단점
- 저장소 크기가 커질 수 있음
- 원본 저장소와의 동기화가 복잡할 수 있음

# 주요 Git 서브모듈 명령어

## 서브모듈 추가
- `git submodule add <저장소 URL> <로컬 경로>`: 새로운 서브모듈 추가

## 서브모듈 초기화 및 업데이트
- `git submodule init`: 서브모듈 초기화
- `git submodule update`: 서브모듈을 지정된 커밋으로 업데이트
- `git submodule update --remote`: 모든 서브모듈을 최신 커밋으로 업데이트

## 서브모듈 상태 확인
- `git submodule status`: 서브모듈의 현재 상태 확인

## 서브모듈 제거
- `git submodule deinit <서브모듈 경로>`: 서브모듈 설정 제거
- `git rm <서브모듈 경로>`: 서브모듈 디렉토리 삭제

## 서브모듈이 포함된 프로젝트 클론
- `git clone --recursive <저장소 URL>`: 서브모듈을 포함하여 저장소 클론

# 주요 Git 서브트리 명령어

## 서브트리 추가
- `git subtree add --prefix=<로컬 경로> <저장소 URL> <브랜치> --squash`: 새로운 서브트리 추가 (--squash는 히스토리를 단일 커밋으로 압축)

## 서브트리 업데이트
- `git subtree pull --prefix=<로컬 경로> <저장소 URL> <브랜치> --squash`: 서브트리 업데이트
- `git subtree push --prefix=<로컬 경로> <저장소 URL> <브랜치>`: 서브트리 변경사항을 원격 저장소로 푸시

## 서브트리 분할
- `git subtree split --prefix=<로컬 경로> -b <새 브랜치>`: 서브트리를 별도의 브랜치로 분리

## 서브트리 비교
- `git diff-tree -p <커밋> <로컬 경로>`: 특정 커밋의 서브트리 변경사항 확인

## 서브트리 사용 팁
- 서브트리는 별도의 초기화가 필요 없음
- 기존 저장소의 모든 히스토리가 포함되므로 저장소 크기가 커질 수 있음
- 원본 저장소와의 동기화는 pull/push 명령으로 관리