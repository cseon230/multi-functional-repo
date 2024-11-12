# 서브모듈 추가
git submodule add https://github.com/junsun0708/jyh-projectB-git-submodule.git repos/jyh/jyh-projectB-git-submodule  
git submodule add https://github.com/junsun0708/jyh-projectA-git-submodule.git repos/jyh/jyh-projectA-git-submodule

# 서브모듈 초기화 및 업데이트
git submodule init
git submodule update

# 모든 서브모듈 업데이트 (각각의 최신 커밋 가져오기)
git submodule update --remote

