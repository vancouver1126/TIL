# Git & GitHub 학습 정리 (2026-11-17)

> 과정 목표: Git과 GitHub를 활용해 코드를 안전하게 관리하고 협업할 수 있는 기본 역량을 갖춘다.

## 목차
1. [버전 관리의 필요성](#1-버전-관리의-필요성)
2. [Git 기본 개념](#2-git-기본-개념)
3. [Git 설치와 초기 설정](#3-git-설치와-초기-설정)
4. [로컬에서 Git 사용하기](#4-로컬에서-git-사용하기)
5. [브랜치 이해하기](#5-브랜치-이해하기)
6. [오늘 실습 기록](#6-오늘-실습-기록)
7. [트러블슈팅](#7-트러블슈팅)
8. [명령어 요약](#8-명령어-요약)

---

## 1. 버전 관리의 필요성

### 버전 관리(Version Control)란?
파일이나 코드의 **변경 이력을 체계적으로 기록하고 관리**하는 것.
언제, 누가, 무엇을 바꿨는지 알 수 있고, 필요하면 과거 상태로 되돌릴 수 있다.

**필요한 이유**: 변경 이력 추적, 실수 복구, 여러 사람이 충돌 없이 협업

**활용 예시**
- 기능을 추가했다가 버그가 생기면 이전 정상 버전으로 바로 되돌린다.
- 팀원이 같은 파일을 동시에 수정해도 누가 무엇을 바꿨는지 확인하고 충돌을 해결한다.
- `최종_진짜최종_v3` 같은 파일 없이 변경 과정을 깔끔하게 관리한다.

### Git vs. 파일 복사

| 구분 | 파일 복사 방식 | Git 사용 |
|---|---|---|
| 파일 관리 | `final.py`, `final_v2.py`, `진짜최종.py`처럼 늘어남 | 변경 내용이 **커밋 단위**로 기록됨 |
| 변경 기록 | 변경 이유와 내용이 남지 않음 | 변경 이력과 책임이 명확함 |
| 되돌리기 | 어려움 | 언제든 특정 시점으로 되돌릴 수 있음 |
| 협업 | 어떤 파일이 최신인지 알기 어려움 | 여러 사람이 동시에 작업 가능 |

### Git vs. GitHub

| Git | GitHub |
|---|---|
| 버전 관리를 위한 **도구(프로그램)** | Git 저장소를 올려두는 **웹 서비스** |
| 내 컴퓨터에서 동작 | 코드 공유 및 협업 플랫폼 |
| | Pull Request, Issue, Review 기능 제공 |

> **Git은 버전 관리 엔진, GitHub는 협업과 공유를 위한 서비스**

---

## 2. Git 기본 개념

### Repository (레포지토리)
파일과 그 변경 이력을 함께 관리하는 저장소. 프로젝트 전체와 모든 버전 기록이 들어 있다.

- **로컬 저장소**: 내 컴퓨터에 있는 저장소
  - 프로젝트 폴더로 이동 → `git init` 실행 → 해당 폴더가 Git 레포지토리가 됨
- **원격 저장소**: GitHub 같은 서버에 있는 저장소
  - GitHub에서 새 Repository 생성 → `git clone`으로 내 컴퓨터에 복사

### Commit (커밋)
변경 사항을 하나의 기록으로 저장하는 단위 (= **하나의 버전**)
- "무엇을 왜 바꿨는지"를 메시지로 남긴다.
- 특정 시점의 프로젝트 상태를 저장한다.
- 되돌아갈 수 있는 **기준점** 역할을 한다.

```bash
git commit -m "회원가입 기능 추가"
```

### Branch (브랜치)
독립적으로 작업할 수 있는 작업 공간
- 기존 코드를 건드리지 않고 기능을 개발한다.
- 여러 기능을 동시에 개발할 수 있다.
- 작업이 끝나면 병합(merge)한다.

```bash
git branch feature/login   # 브랜치 생성
git switch feature/login   # 브랜치 전환
```

### HEAD (헤드)
**현재 내가 작업 중인 위치를 가리키는 포인터**
- 보통 현재 브랜치의 최신 커밋을 가리킨다.
- HEAD가 움직인다 = 작업 기준이 바뀐다.

```
version 1 ── version 2 ── version 3
                              ↑
                            HEAD
```

### Working Directory → Staging Area → Repository

| 단계 | 설명 | 이동 명령어 |
|---|---|---|
| **Working Directory** | 실제로 파일을 수정하고 작업하는 공간. 아직 Git에 기록되지 않은 변경 사항이 있는 상태 | — |
| **Staging Area** | 다음 커밋에 포함할 변경 사항을 미리 모아두는 공간 | `git add` |
| **Repository** | 커밋으로 기록된 버전이 저장되는 공간 | `git commit` |

> 택배로 비유하면 `git add`는 상자에 물건 담기, `git commit`은 상자를 봉인하고 송장 붙이기.
> add 단계가 따로 있어서 여러 파일 중 **원하는 변경만 골라서** 커밋할 수 있다.

---

## 3. Git 설치와 초기 설정

### Git 설치

| OS | 설치 방법 |
|---|---|
| macOS | `brew install git` |
| Windows | Git for Windows 설치 (x64 Setup) |

```bash
git --version   # 설치 확인
```

### GitHub 회원가입
https://github.com/ 에서 회원가입 후 name, email 기록

### git config
Git의 기본 동작을 설정하는 명령어

```bash
# 전역 설정: 모든 저장소에 적용
git config --global user.name "홍길동"
git config --global user.email "hong@example.com"

# 로컬 설정: 특정 저장소에만 적용 (회사/개인 프로젝트 계정 구분 시)
git config user.name "홍길동"
git config user.email "hong@example.com"

# 설정 확인
git config --global --list
```

> `git config`는 GitHub에 로그인하는 명령어가 아니라, **내 컴퓨터에 "커밋 작성자 정보"를 적어두는 설정**이다.
> 이메일 노출이 걱정되면 GitHub Settings → Emails의 **noreply 주소**를 쓴다.

---

## 4. 로컬에서 Git 사용하기

### git init
현재 폴더를 Git 레포지토리로 초기화
- `.git` 숨김 폴더가 생성된다.
- 현재 폴더를 기준으로 버전 관리를 시작한다.

### git status
현재 레포지토리의 상태 확인: 수정된 파일, 스테이징 여부, 커밋 가능한 변경 사항

| 표시 | 의미 |
|---|---|
| `Untracked files` (빨강) | Git이 아직 추적하지 않는 새 파일 |
| `Changes to be committed` (초록) | `git add`로 스테이징된 변경 |
| `M` | Modified, 커밋 이후 수정됨 |
| `D` | Deleted, 커밋된 파일이 폴더에서 삭제됨 |

### git add
변경된 파일을 Staging Area로 올리는 명령어
- 커밋에 포함할 변경 사항을 **선택**하는 단계
- 아직 커밋이 생성된 것은 아님

```bash
git add 파일명   # 특정 파일만
git add .        # 현재 폴더의 모든 변경
```

### git commit
Staging Area에 있는 변경 사항을 하나의 버전으로 기록
- 되돌릴 수 있는 기준점 역할 (게임의 save file)

```bash
git commit -m "커밋 메시지"
```

### 커밋 메시지 작성 규칙
좋은 커밋 메시지는 무엇을 했는지 한눈에 알 수 있어야 한다.

- 명확하고 간결하게 작성
- **동사로 시작**
- **한 커밋 = 한 작업**

```
Add login API
Fix password validation bug
Refactor user service
```

### .gitignore
버전 관리에서 제외할 파일을 지정하는 설정 파일
- 불필요한 파일이 커밋되는 것을 방지
- 환경 변수 파일, 빌드 결과물, 개인 설정 파일 등은 보통 제외

```gitignore
.env
__pycache__/
*.log
```

> **주의**: 이미 커밋된 파일은 Git이 계속 추적한다. 추적을 끊으려면 `git rm --cached 파일명`

### 퀴즈 복습

| 질문 | 정답 |
|---|---|
| Git에서 하나의 작업 단위를 기록한 것은? | **commit** |
| Git을 사용하는 가장 큰 목적은? | **코드 변경 이력을 관리하기 위해** |
| Working Directory와 Staging Area의 차이는? | **스테이징은 커밋할 파일을 선별하는 공간이다** |

---

## 5. 브랜치 이해하기

### Branch (브랜치)
기존 작업과 분리된 **독립적인 작업 흐름**
- 기능 개발, 실험, 수정 작업을 안전하게 진행할 수 있다.
- 협업과 안정적인 배포를 위해 브랜치를 잘 관리하는 것이 중요하다.

```bash
git branch              # 브랜치 목록 확인 (* = 현재 브랜치)
git branch feature/ui   # 브랜치 생성
git switch feature/ui   # 브랜치 전환
git switch -c feat/api  # 생성 + 전환을 한 번에
```

> 브랜치는 폴더가 따로 생기는 게 아니다. **폴더는 하나**이고, `switch`할 때마다 그 브랜치의 마지막 커밋 상태로 폴더 내용이 바뀐다.
> 단, **커밋하지 않은 변경**과 **add하지 않은 새 파일**은 브랜치를 바꿔도 그대로 따라온다.

### 브랜치 관리 전략
어떻게 브랜치를 나누고 합칠지에 대한 약속

| 브랜치 | 역할 |
|---|---|
| `main` | 배포 가능한 안정 버전 |
| `dev` | 개발 중인 기능을 테스트하는 버전 |
| `feature/*` | 기능 개발 |
| `hotfix/*` | 긴급 수정 |

### git merge
다른 브랜치의 작업 내용을 **현재 브랜치로** 합치는 명령어

```bash
git switch main
git merge feature/ui
```

### Fast-forward vs. 3-way merge

| 구분 | 상황 | 동작 |
|---|---|---|
| **Fast-forward** | 브랜치가 직선으로 이어질 때 | 커밋 포인터만 앞으로 이동 |
| **3-way merge** | 브랜치가 갈라졌다가 다시 합쳐질 때 | **병합 커밋**이 새로 생성됨 |

> 분기가 없으면 Fast-forward, 분기가 있으면 3-way merge

```
[Fast-forward]
1 ── 2 ── A ── B
     ↑         ↑
   main     feature/ui
→ 병합하면 main 포인터가 B로 이동만 함

[3-way merge]
main ── 1 ── 2 ──────── 3 (병합 커밋)
              \        /
               feature/ui
```

### Merge conflict 해결
같은 파일의 **같은 부분**을 서로 다르게 수정하면 충돌이 발생한다.
Git은 자동 병합을 시도하고, 판단이 필요한 경우에만 충돌로 알려준다.

**해결 방법**
1. 충돌 파일 확인
2. 충돌 코드 직접 수정
3. 새로운 커밋 생성

---

## 6. 오늘 실습 기록

### 1) 환경 설정 (macOS)
```bash
# Homebrew 설치
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew --version

# Git 설치
brew install git

# Homebrew 경로를 PATH 맨 앞에 등록
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"

# 확인
which git        # /opt/homebrew/bin/git
git --version    # git version 2.56.1

# 사용자 정보 등록
git config --global user.name "vancouver1126"
git config --global user.email "<GitHub noreply 이메일>"
```

### 2) 레포지토리 생성과 첫 커밋
```bash
mkdir -p ~/project/oz/ai-08/git
cd ~/project/oz/ai-08/git
git init
git branch -m master main   # 기본 브랜치 이름을 main으로

# hello.py 작성 후
git add hello.py
git commit -m "Create hello.py"

# world.py 작성 후
git add world.py
git commit -m "Create world.py"

git log   # 커밋 기록 확인 (q로 종료)
```

### 3) 브랜치 실습
```bash
git switch -c feat/api   # feat/api 브랜치 생성 + 이동
# api.py 작성 후
git add api.py
git commit -m "Create api.py"

git switch main          # main으로 이동하면 api.py가 사라짐
git switch feat/api      # 다시 이동하면 api.py가 나타남
```

### 4) 실습 결과 커밋 히스토리
```
* 0e178bc Create api.py
* 774428f Fill hello.py and world.py
* 67ac69b Create api.py
* 7492d19 Create api.py
* 160005f Create world.py
* 882e592 Create hello.py
```

| 파일 | 내용 |
|---|---|
| `hello.py` | `print('hello')` |
| `world.py` | `print('world')` |
| `api.py` | `print("api")` |

---

## 7. 트러블슈팅

| 증상 | 원인 | 해결 |
|---|---|---|
| `git --version`이 `2.50.1 (Apple Git)`로 나옴 | Apple 기본 git이 PATH에서 먼저 잡힘 | `brew shellenv` 설정을 `~/.zprofile`에 추가 |
| `zsh: command not found: #` | `#` 주석 줄까지 같이 붙여넣음 | 무시해도 됨. 명령어 줄만 붙여넣기 |
| `error: key does not contain a section` | `git config --global` 뒤에 `user.name` 누락 | `git config --global user.name "이름"` |
| `fatal: not a git repository` | `git init` 하지 않은 폴더에서 실행 | 실습 폴더로 `cd` 후 `git init` |
| `>>> git status` → `SyntaxError` | 터미널이 파이썬 실행 모드(`>>>`) | `exit()` 또는 `Ctrl + D`로 나오기 |
| 커밋 결과가 `0 insertions(+)` | VS Code에서 **저장하지 않은 채** 커밋 | `Cmd + S` 저장 후 add/commit. 탭의 ● 표시 확인. File → Auto Save 권장 |
| `git switch feat/api` 실패 | 존재하지 않는 브랜치 | `git switch -c feat/api`로 생성하며 이동 |
| 프롬프트 앞 `(base)` | Anaconda 기본 환경이 자동 활성화 | `conda deactivate` 또는 `conda config --set auto_activate_base false` |

---

## 8. 명령어 요약

| 명령어 | 설명 |
|---|---|
| `git --version` | 설치 확인 |
| `git config --global user.name "이름"` | 작성자 이름 설정 |
| `git config --global user.email "이메일"` | 작성자 이메일 설정 |
| `git init` | 현재 폴더를 Git 레포지토리로 초기화 |
| `git status` | 현재 상태 확인 |
| `git add 파일명` / `git add .` | Staging Area에 올리기 |
| `git commit -m "메시지"` | 커밋 생성 |
| `git log` | 커밋 기록 보기 |
| `git branch` | 브랜치 목록 |
| `git branch 이름` | 브랜치 생성 |
| `git switch 이름` | 브랜치 전환 |
| `git switch -c 이름` | 브랜치 생성 + 전환 |
| `git merge 이름` | 해당 브랜치를 현재 브랜치에 병합 |
| `git rm --cached 파일명` | 추적 중단 (파일은 유지) |
