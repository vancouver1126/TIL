# Git 병합 & GitHub 협업 학습 정리 (2026-11-18)

> 어제(11-17) 배운 Git 기본기 위에, 오늘은 **브랜치 병합 방식**, **충돌 해결**, **원격 저장소(GitHub) 연동과 협업**, **자주 쓰는 명령어**를 다뤘다.

## 목차
1. [병합(Merge)의 원리: 2-way vs 3-way](#1-병합merge의-원리-2-way-vs-3-way)
2. [Fast-forward와 3-way merge](#2-fast-forward와-3-way-merge)
3. [병합 옵션: --ff / --no-ff / --ff-only](#3-병합-옵션---ff----no-ff----ff-only)
4. [Merge conflict 해결](#4-merge-conflict-해결)
5. [GitHub 기초: remote, push, pull](#5-github-기초-remote-push-pull)
6. [협업: Clone vs Fork, Pull Request, Code Review, Issue](#6-협업-clone-vs-fork-pull-request-code-review-issue)
7. [자주 사용하는 명령어](#7-자주-사용하는-명령어)
8. [오늘 실습 기록](#8-오늘-실습-기록)
9. [트러블슈팅](#9-트러블슈팅)
10. [명령어 요약](#10-명령어-요약)

---

## 1. 병합(Merge)의 원리: 2-way vs 3-way

두 브랜치를 합칠 때 Git이 "어느 쪽 내용을 쓸지" 판단하는 방식의 차이다.

| 구분 | 비교 대상 | 판단 능력 |
|---|---|---|
| **2-way merge** | 두 브랜치의 최신 버전 **2개**만 비교 | 내용이 다르다는 것만 알고, **누가 바꿨는지는 모른다** → 다른 곳마다 사람이 골라야 함 |
| **3-way merge** | 두 브랜치의 최신 버전 + **공통 조상(merge base)**, 총 **3개** 비교 | 공통 조상과 비교해 **어느 쪽이 바꿨는지 안다** → 한쪽만 바꾼 부분은 자동으로 반영 |

### 3-way merge의 판단 규칙

| 공통 조상 | main | feature | 결과 |
|---|---|---|---|
| A | A | A | A (변경 없음) |
| A | A | **B** | **B** (feature만 변경 → 자동 반영) |
| A | **C** | A | **C** (main만 변경 → 자동 반영) |
| A | **C** | **B** | ⚠️ **충돌** (양쪽 모두 같은 부분을 변경) |

> Git은 **3-way merge**를 사용한다. 그래서 대부분은 자동으로 합쳐지고, **같은 부분을 양쪽에서 다르게 고친 경우에만** 충돌로 알려준다.

---

## 2. Fast-forward와 3-way merge

### Fast-forward merge
- 브랜치가 **직선으로 이어질 때** (main에 새 커밋이 없을 때)
- 새 커밋 없이 **main 포인터만 앞으로 이동**

```
병합 전                      병합 후 (git merge feature)
1 ── 2 ── A ── B             1 ── 2 ── A ── B
     ↑         ↑                            ↑
   main     feature                   main, feature
```

### 3-way merge
- 브랜치가 **갈라졌다가 다시 합쳐질 때** (양쪽에 각자 커밋이 있을 때)
- 공통 조상 기준으로 합친 **병합 커밋(merge commit)** 이 새로 생성됨

```
        A ── B   (feature)
       /      \
1 ── 2 ── C ── M   (main, M = 병합 커밋)
```

> **분기가 없으면 Fast-forward, 분기가 있으면 3-way merge**

---

## 3. 병합 옵션: --ff / --no-ff / --ff-only

`git merge`는 상황에 따라 Fast-forward와 3-way 중 하나를 자동으로 고른다. 옵션으로 이 동작을 강제할 수 있다.

| 옵션 | Fast-forward 가능할 때 | 분기가 있을 때 | 용도 |
|---|---|---|---|
| `--ff` (기본값) | 포인터만 이동 | 병합 커밋 생성 | 일반적인 병합 |
| `--no-ff` | **병합 커밋을 강제로 생성** | 병합 커밋 생성 | 브랜치 작업 이력을 남기고 싶을 때 |
| `--ff-only` | 포인터만 이동 | **병합을 거부하고 중단** | 히스토리를 일직선으로만 유지하고 싶을 때 |

### --no-ff (No Fast-forward)
```bash
git switch main
git merge --no-ff feature
```
- 직선으로 이어져 있어도 **병합 커밋을 만든다.**
- `git log --graph`에 "이 기능은 feature 브랜치에서 개발해서 합쳤다"는 흔적이 남는다.
- 기능 단위로 이력을 관리하거나, 나중에 병합 커밋 하나만 `revert`해서 기능 전체를 되돌리기 쉽다.

```
--ff (기본)                     --no-ff
1 ── 2 ── A ── B (main)        1 ── 2 ──────── M (main)
                                     \        /
                                      A ── B   (feature)
```

### --ff-only (Fast-forward only)
```bash
git switch main
git merge --ff-only feature
```
- Fast-forward가 가능할 때만 병합한다.
- 분기가 있으면 병합 커밋을 만들지 않고 이렇게 중단한다.
  ```
  fatal: Not possible to fast-forward, aborting.
  ```
- 의도치 않은 병합 커밋이 생기는 걸 막아준다. `git pull --ff-only`로 원격 변경을 안전하게 받을 때 자주 쓴다.

### 기본 동작 설정
```bash
git config --global merge.ff false    # merge할 때 항상 --no-ff
git config --global pull.ff only      # pull할 때 항상 --ff-only
```

---

## 4. Merge conflict 해결

### 충돌이 생기는 조건
같은 파일의 **같은 부분**을 서로 다른 브랜치에서 다르게 수정한 뒤 병합할 때.

### 실습 흐름 (슬라이드)
```bash
# 1. 레포지토리 초기화와 첫 커밋
git init
echo "hello" > test.txt
git add .
git commit -m "first commit"

# 2. feature 브랜치에서 수정
git branch feature
git switch feature
echo "hello from feature" > test.txt
git add .
git commit -m "update from feature"

# 3. main에서 같은 줄을 다르게 수정
git switch main
echo "hello from main" > test.txt
git commit -am "update from main"   # -a: 추적 중인 파일을 add + commit 한 번에

# 4. 병합 시도 → 충돌
git merge feature

# 5. 확인 → 수정 → 커밋
cat test.txt
vi test.txt
git commit -am "resolve merge conflict"
git log --oneline --graph
```

### 충돌이 나면 보이는 것
```
CONFLICT (content): Merge conflict in test.txt
Automatic merge failed; fix conflicts and then commit the result.
```

파일 안에 양쪽 내용이 표시와 함께 들어간다.
```
<<<<<<< HEAD
hello from main          ← 현재 브랜치(main)의 내용
=======
hello from feature       ← 합치려는 브랜치(feature)의 내용
>>>>>>> feature
```

### 해결 방법
1. 충돌 파일 확인: `git status` → `both modified: test.txt`
2. 충돌 코드 직접 수정: 최종 내용만 남기고 `<<<<<<<`, `=======`, `>>>>>>>` 줄 삭제
   - VS Code: **Accept Current / Accept Incoming / Accept Both** 버튼으로 선택 가능
3. 저장 후 새로운 커밋 생성: `git add 파일` → `git commit`

> 병합을 아예 취소하고 싶으면 `git merge --abort`

---

## 5. GitHub 기초: remote, push, pull

### 원격 저장소
인터넷에 있는 Git 저장소. 여러 사람이 같은 코드를 공유하고, 백업 역할을 하며, **협업의 기준점**이 된다.
GitHub에서 새 Repository를 만들면 원격 저장소 주소(URL)가 발급된다.

```bash
git remote add origin 원격주소     # 원격 저장소를 origin이라는 이름으로 연결
git remote -v                      # 연결된 원격 저장소 확인
git remote set-url origin 새주소   # 연결 주소 변경
```

### push / pull
로컬과 원격 저장소를 동기화하는 명령어

| 명령어 | 방향 | 설명 |
|---|---|---|
| `git push` | 로컬 → 원격 | 로컬 커밋을 원격 저장소로 업로드 |
| `git pull` | 원격 → 로컬 | 원격 변경 사항을 내려받아 로컬에 반영 (= fetch + merge) |

```bash
git push -u origin main   # -u: 연결을 기억해서 다음부터 git push만 입력
git pull
```

> `git pull`도 내부적으로 merge를 한다. 그래서 `--ff-only`, `--no-ff` 같은 병합 옵션이 똑같이 적용된다.

---

## 6. 협업: Clone vs Fork, Pull Request, Code Review, Issue

### Clone vs. Fork

| Clone | Fork |
|---|---|
| 원격 저장소를 그대로 **내 로컬로** 복사 | 다른 사람의 저장소를 **내 GitHub 계정으로** 복사 |
| 같은 저장소에 **push 권한이 있을 때** | 원본 저장소에 **push 권한이 없을 때** |
| 팀 내부 협업 | 오픈소스 기여 |

### Pull Request (PR)
변경 사항을 원본 저장소에 반영해 달라고 요청하는 것.
변경 내용 설명 → 리뷰 요청 → **병합 전 검증 단계**

### Code Review
코드를 병합하기 전에 함께 확인하는 과정 (버그 예방, 코드 품질 유지, 팀 규칙 공유)
- 코드 중심으로 리뷰
- 이유를 남긴 코멘트
- **승인 후 병합**

### Issue
할 일, 버그 리포트, 기능 요청 등을 관리하는 기능. PR과 Issue를 연결해서 작업을 관리한다.
```
Resolve: #1234     ← PR 설명에 적으면 해당 Issue와 연결
```

### 협업 방식 ① Fork & Clone (권한이 없을 때)
```bash
# 1. GitHub에서 Issue 생성 → 내 계정으로 Fork
git clone 내_포크_저장소_URL                    # 2. 내 포크를 로컬로
git remote add upstream 원본_저장소_URL         # 3. 원본 저장소 연결
git pull upstream main                          # 4. 원본의 최신 코드 받기
git switch -c feature/add-my-name               # 5. 작업 브랜치 생성
# 6. 커밋 → 내 포크에 push → 원본 저장소로 PR 생성
```

### 협업 방식 ② Collaborator (권한이 있을 때)
```bash
# 1. 저장소 Settings에서 Collaborator 등록
# 2. Branch protection rules 추가: "Require a pull request before merging"
#    → main에 직접 push 금지, PR로만 병합
git clone 원본_저장소_URL
git switch -c feature/mybranch
git commit -m "message"
git push origin feature/mybranch
# 3. GitHub에서 feature/mybranch → main PR 생성 → 리뷰 → 병합
```

| 원격 이름 | 가리키는 저장소 |
|---|---|
| `origin` | 내가 clone해 온 저장소 (Fork 방식에선 **내 포크**) |
| `upstream` | Fork의 **원본 저장소** |

---

## 7. 자주 사용하는 명령어

### git show: 커밋 하나 자세히 보기
변경 내용(diff), 커밋 메시지, 작성자, 날짜를 출력한다.
```bash
git show              # = git show HEAD, 현재 브랜치의 최신 커밋
git show 커밋해시      # 특정 커밋
```

### git diff: 두 상태의 차이 비교
```bash
git diff                # Working Directory ↔ Staging Area (아직 add 안 한 변경)
git diff --staged       # Staging Area ↔ HEAD (add했지만 커밋 안 한 변경)
git diff HEAD~1 HEAD    # 커밋 ↔ 커밋
```

### git blame: 줄마다 누가 언제 고쳤는지
```bash
git blame main.py
```
버그 원인을 추적하거나 "이 줄은 왜 이렇게 되어 있지?"를 확인할 때 쓴다.

### git stash: 작업 중인 변경을 임시 보관
커밋하지 않고 작업을 잠시 치워둔다. 브랜치 이동이나 긴급 수정 시 유용하다.
```bash
git stash          # 현재 변경 사항 임시 저장
git stash list     # 저장된 목록 확인
git stash pop      # 가장 최근 것을 다시 적용하고 목록에서 제거
git stash apply    # 목록에서 제거하지 않고 적용만
```

### git reset vs git revert: 커밋 되돌리기

| 구분 | git reset | git revert |
|---|---|---|
| 방식 | 브랜치를 **과거 커밋으로 이동** | 되돌리는 **새 커밋을 생성** |
| 히스토리 | **바뀜** (커밋이 사라짐) | **보존** ("취소했다"는 기록이 남음) |
| 사용처 | 아직 push하지 않은 내 로컬 커밋 | **이미 공유된 커밋**, 협업 중 안전한 롤백 |

```bash
git reset --soft HEAD~1    # 커밋만 취소 (변경 내용은 스테이징된 채 유지)
git reset HEAD~1           # --mixed(기본): 커밋 + 스테이징 취소 (파일 내용은 유지)
git reset --hard HEAD~1    # 커밋 + 스테이징 + 파일까지 전부 되돌림 ⚠️
git revert 커밋해시         # 해당 커밋을 취소하는 새 커밋 생성
```

> `HEAD~1` = HEAD의 바로 이전 커밋. 협업 브랜치에서 `reset`은 다른 사람의 히스토리와 어긋나므로 주의.

### git commit --amend: 마지막 커밋 고치기
가장 최근 커밋을 새 커밋으로 교체한다. 메시지 수정, 빠뜨린 파일 추가에 쓴다.
```bash
git commit --amend              # 마지막 커밋 수정 (메시지 편집기 열림)
git commit --amend --no-edit    # 메시지는 그대로, 파일만 추가 반영
```

---

## 8. 오늘 실습 기록

### 1) 저장소 새로 시작
```bash
cd ~/project/oz/ai-08/git
rm -rf .git                                   # 기존 git 기록 삭제 (파일은 유지)
git init
git branch -m master main                     # 기본 브랜치 이름 변경
git config --global init.defaultBranch main   # 앞으로 git init 시 main으로 시작

git add hello.py world.py
git commit -m "Create hello.py and world.py"
```

### 2) 기능 브랜치 작업과 3-way merge
```bash
git branch feat/ai
git branch feat/ui

git switch feat/ui
# ui.py 수정 후
git add ui.py
git commit -m "Update ui.py"

git switch main
# ai.py 생성 후 main에 커밋 → main과 feat/ui가 갈라짐
git merge feat/ui    # 분기가 있으므로 3-way merge → 병합 커밋 생성
```

### 3) left / right 충돌 실습
```bash
# 공통 커밋에서 브랜치를 먼저 만든다 (수정 전에!)
git branch left
git branch right

git switch left
echo 'print("Hello, left")' > hello.py
git add hello.py
git commit -m "hello, left"

git switch right
echo 'print("Hello, right")' > hello.py
git add hello.py
git commit -m "hello, right"

git switch main
git merge left       # 직선 → Fast-forward
git merge right      # 분기 + 같은 줄 수정 → CONFLICT

# hello.py에서 최종 내용 선택 후 저장
git add hello.py
git commit -m "Fix conflict"
```

### 4) 실습 결과 커밋 그래프
```
*   06f2000 (HEAD -> main) Fix conflict
|\
| * 33c337d (right) hello, right
* | e7ffaaa (left) hello, left
|/
*   97594cc Merge branch 'feat/ui'
|\
| * 86000c8 (feat/ui) Update ui.py
* | 1cd010c Create ai.py
|/
* 80da565 Create ui.py
* c8c0c38 Create files
* 45ccc51 Create hello.py and world.py
```
- `Merge branch 'feat/ui'`: 분기가 있어서 생긴 **3-way merge의 병합 커밋**
- `hello, left`: main 기준 직선이라 **Fast-forward**로 합쳐짐 (병합 커밋 없음)
- `Fix conflict`: right와 충돌을 해결하며 만든 **병합 커밋**

### 5) 최종 파일 상태

| 파일 | 내용 |
|---|---|
| `hello.py` | `print("Hello, left")` (충돌 해결 시 left 선택) |
| `world.py` | `print('world')` |
| `api.py` | `print("api")` |
| `ai.py` | `print("AI")` |
| `ui.py` | `print("UI!!!!!!!!")` |

### 6) GitHub에 올리기
```bash
git remote add origin https://github.com/vancouver1126/저장소이름.git
git remote -v
git push -u origin main
```

---

## 9. 트러블슈팅

| 증상 | 원인 | 해결 |
|---|---|---|
| `zsh: permission denied: /Users/mac/project/...` | 경로 앞에 `cd`를 빠뜨려서 폴더를 "실행"하려 함 | `cd ~/project/oz/ai-08/git` |
| `git branch`가 아무것도 출력 안 함 | `git init` 직후 커밋이 없어서 브랜치가 아직 실제로 없음 | 첫 커밋을 하면 생김 |
| `현재 브랜치 master` | `git init` 기본 브랜치 이름이 master | `git branch -m master main`, `init.defaultBranch main` 설정 |
| `.git` 삭제 후 `feat/api` 브랜치가 사라짐 | 브랜치와 커밋 기록은 전부 `.git` 안에 저장됨 | 브랜치 다시 생성: `git switch -c 브랜치명` |
| `커밋할 사항 없음, 작업 폴더 깨끗함` | VS Code에서 고친 내용을 **저장하지 않음** | `Cmd + S` 후 `cat 파일`로 확인 → add → commit |
| 충돌이 안 나고 그냥 합쳐짐 | 한쪽을 먼저 커밋한 **뒤에** 다른 브랜치를 만들어 분기가 없음 | 공통 커밋에서 **두 브랜치를 먼저 만들고** 각각 수정 |
| `! [rejected] main -> main (fetch first)` | 원격 저장소에 내 로컬에 없는 커밋이 있음 (다른 프로젝트 저장소에 push 시도) | 실습용 새 저장소를 만들어 `git remote set-url origin 새주소`. **`git push -f` 금지** (원격 파일이 삭제됨) |

---

## 10. 명령어 요약

| 명령어 | 설명 |
|---|---|
| `git merge 브랜치` | 해당 브랜치를 현재 브랜치에 병합 |
| `git merge --no-ff 브랜치` | Fast-forward 가능해도 병합 커밋 생성 |
| `git merge --ff-only 브랜치` | Fast-forward 가능할 때만 병합, 아니면 중단 |
| `git merge --abort` | 충돌 중인 병합 취소 |
| `git commit -am "메시지"` | 추적 중인 파일 add + commit 한 번에 |
| `git log --oneline --graph --all` | 전체 브랜치 히스토리를 그래프로 보기 |
| `git remote add origin URL` | 원격 저장소 연결 |
| `git remote -v` | 연결된 원격 저장소 확인 |
| `git remote set-url origin URL` | 원격 저장소 주소 변경 |
| `git push -u origin main` | 로컬 커밋 업로드 + 연결 기억 |
| `git pull` | 원격 변경 내려받아 병합 |
| `git clone URL` | 원격 저장소를 로컬로 복사 |
| `git show` | 커밋 상세 보기 |
| `git diff` / `git diff --staged` | 변경 내용 비교 |
| `git blame 파일` | 줄별 수정 이력 |
| `git stash` / `git stash pop` | 작업 임시 보관 / 복원 |
| `git reset --soft/--mixed/--hard HEAD~1` | 커밋 되돌리기 (히스토리 변경) |
| `git revert 커밋해시` | 커밋 취소용 새 커밋 생성 (히스토리 보존) |
| `git commit --amend` | 마지막 커밋 수정 |
