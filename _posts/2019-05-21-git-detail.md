---
layout: post
title:  "[Git] Git into the Git"
date:   2019-05-21 09:00:01
categories: tools
---



# 1. Git 개념 정리

지금까지 Git의 기본 명령어들을 활용하여 변경 사항을 원격 저장소와 로컬 저장소에 반영하는 방법을 배웠다. 하지만 아직 명령어들을 개념적으로 이해한 것은 아니다. 여기서는 Git을 이해하는데 필요한 개념들을 설명한다.



## 1.1 파일의 상태

아래 그림을 보자. 위쪽의 사각형은 파일의 상태를 나타내고 화살표에 쓰인 단어는 "edit"을 제외하고 모두 Git의 동작(verb)다. 모든 Git 명령어는 `git <verb> <options>` 형태이므로 마치 `주어-동사(verb)-목적어`의 영어 문장처럼 쓰인다. 그래서 중간의 큰 단위의 동작을 나타내는 인자를 *verb*라 부른다.

![status-cycle](/ian-flow/assets/2019-05-15-git-intro/status-cycle.png)



Git 저장소 안에서 파일은 그림처럼 네 가지 상태를 가질 수 있다.

- **Untracked**: 아직 Git으로 버전 관리되지 않은 상태이다.
- **Staged**: `git add`를 실행하면 *Untracked*나 *Modified* 상태의 파일들이 *Staged*가 된다. Stage 했을 당시의 파일 상태가 이후 commit할 때 저장이 된다. 그래서 add를 한 후에 수정을 하더라도 commit은 현재 상태가 아닌 add 한 당시의 상태를 저장한다.
- **Modified**: Stage 하지 않은 변경 사항이 있는 상태다. *Staged* 상태의 파일을 수정하면 Stage 이후의 변경 사항은 *Modified*에 속해서 하나의 파일이 *Staged* 이면서 동시에 *Modified* 상태가 된다. *Unmodified* 파일을 수정하면 완전히 *Modified* 상태가 된다.
- **Unmodified**: *Staged* 상태의 파일을 commit 하면 *Unmodified* 상태가 된다. Git은 commit 단위로 해쉬(hash)를 할당하고 버전을 관리한다. 즉 언제든 과거에 commit한 상태로 돌아갈 수 있다.



원격 저장소와 교류하는 동작(verb)은 네 가지가 있다.

- **pull**: 원격 저장소의 commit을 로컬 저장소에 다운로드 받고 내용을 합친다.
- **push**: 로컬 저장소의 commit을 원격 저장소로 올려서 합친다. 로컬 저장소에 없는 commit이 원격 저장소에 있다면 먼저 pull을 실행해야 push 할 수 있다.
- **clone**: 원격 저장소를 복사한 로컬 저장소를 만든다.
- **fetch**: pull은 사실 원격 저장소의 commit을 내려받는 fetch와 내용을 합치는 merge 두 명령을 한번에 실행한 것이다. fetch만 하게 되면 현재 로컬 저장소의 파일에는 변화가 생기지 않는다.



## 1.2 저장소 공간

지금까지는 파일의 관점에서 파일의 상태 변화를 알아보았다. 아래 그림은 Git 저장소 내부의 세 가지 **공간**을 나타낸 것이다.

![git_spaces](/ian-flow/assets/2019-05-15-git-intro/git_spaces.png)

- **작업 트리, Work Tree**: 현재 사용자에게 보이는 파일들이 있는 공간이다. 이곳에는 다양한 상태의 파일들이 섞여있다.
- **인덱스, Index**: 가장 최근 commit에 Stage 한 내용까지 반영된 공간이다. 파일을 add 하면 그 당시의 상태가 이곳으로 복사된다. commit 하기 전까지 add 한 변경 사항이 이곳에 쌓인다.
- **저장소, Repository**: commit을 하게 되면 Staged 파일들이 새로운 commit으로 저장되고 HEAD가 새 commit으로 변경된다. 저장소에는 commit들이 쌓인다.
- **HEAD**: 그림에서 저장소 대신 HEAD가 쓰이기도 하는데 HEAD는 현재 작업하는 로컬 브랜치의 최신 commit을 가리키는 포인터다.



# 2. Git Verb 정리

Git은 수천가지의 기능이 있는데 다양한 기능을 큰 분류로 묶어서 동사로 나타낸 것이 *verb*다. Git의 첫 번째 인자로 들어가서 대략의 동작을 말하고 이후 세부 옵션으로 정확한 동작을 기술한다. 여기서는 각 verb의 다양한 기능 중에서 자주 쓰이는 것 위주로 용법을 정리한다.



## 2.1. 로컬 저장소 작업

로컬 저장소에서 버전을 저장하며 작업하는데 필요한 명령어들이다.



### git init

> 현재 디렉토리를 Git이 관리하는 저장소로 초기화 한다.

------

### git config

> Git에 대한 설정을 변경할 수 있는 명령어. 설정할 수 있는 변수가 수백가지지만 대부분은 사용자 등록 정도만 사용한다.
>
> `git config [--global] user.name <name>` : 사용자 이름을 등록한다. `--global` 옵션을 쓰면 이 PC의 모든 로컬 저장소에 기본 사용자가 된다. `--global`을 빼면 현재 저장소에만 적용이 된다.
>
> `git config [--global] user.email <email>`: 사용자의 이메일을 등록한다. `--global`의 용도는 위와 같다.
>
> `git config [--global] --list` : Git의 모든 설정 정보를 조회한다. `--global`을 쓰면 `--global` 옵션을 주고 설정한 전역 설정 정보를 조회한다.

------

### git status

> HEAD로부터 변경 사항이 있는 파일들을 상태별로 묶어서 보여준다. Untracked, Not staged, Staged, Changes to be commited 등의 상태가 있다. 그리고 현재 상태에서 쓸만한 명령어 추천해준다.

------

### git add

> 지정한 파일(들)의 최신 상태를 인덱스에 업데이트한다. Untracked나 Modified 상태의 파일을 stage하여 다음 commit에 변경사항이 들어갈 수 있게 준비한다.
>
> `git add <filename>` : 특정 파일을 stage 한다.
>
> `git add <pattern like *.txt>` : 현재 디렉토리에서 패턴과 일치하는 모든 파일을 stage 한다.
>
> `git add .` : 현재 디렉토리와 하위 디렉토리의 모든 변경된 파일들을 stage 한다. 하지만 파일을 삭제한 사실을 stage 해주진 않는다.
>
> `git add -A`: 현재 디렉토리와 하위 디렉토리의 모든 변경된 파일들과 삭제된 파일들을 stage 한다.

------

### git commit

> 현재 인덱스 상태를 저장소에 저장한다. commit을 하면 stage한 상태까지를 저장하고 hash 혹은 checksum을 부여한다. (checksum과 hash는 비슷하게 쓰이지만 방식이 다르고 git에서는 checksum이 더 정확한 용어지만 hash라고 많이 부른다.)  Hash는 코드로부터 자동으로 생성되는 40자리 문자로서 commit의 ID 같은 역할을 한다. Hash는 `git log` 명령어를 통해 확인할 수 있다. 이 hash를 이용해 나중에 언제든 예전에 commit한 상태로 돌아갈 수 있다.
>
> `git commit -m <message>` : 현재 stage된 변경사항을 message, author, hash와 함께 저장한다.  `-m` 옵션이 없으면 Git 기본 에디터가 실행되서 그곳에서 메시지를 작성하게 한다. 그냥 커맨드에서 -m 옵션을 써서 메시지를 입력하는게 낫다.
>
> `git commit --amend -m <message>` : 직전 commit에서 빠진게 있을 때 변경 사항을 추가하고 add 한 뒤 `--amend` 옵션을 붙여 commit하면 직전 commit을 없애고 추가 변경 사항까지 합친 새로운 commit을 만든다. 변경 사항이 없더라도 단순히 commit message를 다시 쓰고자 할 때도 사용된다.

------

### git rm

> rm은 remove의 약자로 파일을 삭제하고 삭제한 상태를 stage한다. 즉 파일을 삭제한 후 `git add -A` 한 것과 같다.  Git으로 버전 관리되는 파일은 가급적 rm을 이용해 삭제하는 것이 좋다.
>
> `git rm <file_name>` : 지정한 파일을 삭제하고 stage한다.
>
> `git rm <file_pattern>` : 패턴과 일치하는 모든 파일을 삭제하고 stage한다. 예를들어 `git rm *.txt` 라고 하면 모든 텍스트 파일을 삭제하는 것이다.
>
> `git rm -rf <dir_name>` : 디렉토리를 삭제할 때는 -rf 옵션을 줘야한다.
>
> `git rm --cached <file_name>` : 실제 파일은 삭제하지 않고 파일을 인덱스에서 제외하여 Untracked 상태로 만든다.

------

### git mv

> mv는 move의 약자로 파일을 이동하고 이동한 상태를 stage한다. 버전 관리되는 파일을 A 디렉토리에서 B 디렉토리로 옮겨버리면 A에서는 파일이 삭제되고 B에는 새 파일이 추가된 것으로 인식한다. 이를 stage하기 위해서는 역시 `git add -A`를 해줘야한다. 그러므로 파일을 이동할 때는 가급적 mv를 이용하는 것이 좋다. 
>
> mv는 또한 단순히 파일의 이름을 바꾸는데도 사용된다. 같은 경로에 다른 이름으로 옮기면 파일명 변경이 된다.
>
> `git mv <src_file> <dst_file>` : src_file을 dst_file로 이름을 바꾼다.
>
> `git mv <src_file> <dst_dir>` : src_file을 dst_dir 디렉토리로 옮긴다.
>
> `git mv <src_file> <dst_dir/dst_file>` : src_file을 dst_dir 디렉토리 아래 dst_file이란 이름으로 옮긴다.

------



## 2.2 원격 저장소 사용

원격저장소와 데이터를 주고 받으며 활용하는데 필요한 명령어들이다.



### git remote

> 로컬 저장소와 연결된 원격 저장소들을 관리한다.
>
> `git remote -v` : 현재 가지고 있는 원격 저장소 목록 보여주기
>
> `git remote add <name>  <url>` : 원격 저장소 주소 url을 name이란 별명으로 등록
>
> `git remote rename <old_name>  <new_name>` : 원격 저장소 별명을 "old_name"에서 "new_name"으로 변경
>
> `git remote remove <name>` : 원격 저장소 삭제
>
> `git remote set-url <name> <new_url>`: name 원격 저장소의 주소를 new_url로 수정

---

### git clone

> 원격 저장소를 복사한 로컬 저장소를 만든다. 저장소를 복사한다는 의미는 모든 branch의 모든 commit을 다운로드 받는다는 것이다. GitHub에서 "Download Zip"으로도 코드를 받을 수 있지만 과거 이력없이 최신 상태의 코드만 받게 된다.
>
> `git clone <repository_url> [dir_name]` : `repository_url` 주소의 원격 저장소를 복사한다. 기본적으로는 저장소 이름과 같은 디렉토리가 생기고 뒤에 `dir_name`을 지정하면 그 이름으로 디렉토리가 생긴다.
>
> `git clone -o <name> <repository_url>` : -o 옵션을 통해 원격 저장소의 이름을 `origin`이 아닌 다른 이름으로 지정할 수 있다.

---

### git pull

> 원격 저장소의 새로운 변경 사항(commit)들을 로컬 저장소에 내려받고 작업 트리에 그 내용을 반영한다. Pull을 실행하기 전에 반드시 로컬 저장소의 상태는 모든 것이 commit이 된 "Unmodified" 상태여야 pull을 할 수 있다. Pull은 사실 모든 commit을 내려받는 `git fetch`와 내려받은 commit들과 현재 로컬 파일에 반영하는 (합치는) `git merge FETCH_HEAD` 두 명령어를 결합한 것이다. 따라서 pull에는 merge와 관련된 옵션들이 있다.
>
> `git pull` : 원격 저장소의 모든 branch의 commit들을 로컬 저장소에 받고 각 branch를 모두 merge 한다. 원격의 master는 로컬의 master와 합치고 원격의 some_branch는 로컬의 some_branch와 합친다.
>
> `git pull <remote> <local_branch>` : local_branch만 변경 사항(commit)을 내려받고 합친다.
>
> `git pull [--ff / --no-ff / --only-ff]` : merge 방식에 fast-forward 방식과 non-fast-forward 방식이 있는데 두 방식에 대한 설명은 [이곳](<https://backlog.com/git-tutorial/kr/stepup/stepup1_4.html>)에서 확인할 수 있다. `--only-ff`는 fast-forward 방식이 가능할 때만 merge를 하라는 것이다.

---

### git push

> 로컬 저장소의 새로운 commit을 원격 저장소로 올린다. 현재 로컬 저장소의 파일 상태나 Stage 여부에 상관없이 오직 commit에 들어간 변경 사항만 원격 저장소로 올린다.
>
> `git push origin master` : Git 초보자들이 가장 많이 쓰는 명령어 중 하나이다. 저장소를 clone 받으면 `master`라는 기본 branch가 선택되고 원격 저장소는 자동으로 `origin`이란 이름으로 저장된다. 그래서 `master` branch에서 작업 후 commit을 업로드 할 때 이 명령어를 쓰게된다.
>
> `git push <remote_repository> <local_branch>` : local_branch의 commit들을 원격 저장소의 같은 이름의 branch에 올린다. 예를 들어 `git push origin master`는 로컬 저장소의 `master` branch의 commit들을 원격 저장소의 `master` branch(origin/master)에 올린다는 것이다. 원격 저장소에 local_branch가 없을 경우 GitHub에서 자동으로 같은 이름의 branch를 만들어준다.
>
> `git push <remote_repository> <local_branch>:<remote_branch>` : local_branch의 commit들을 원격 저장소의 remote_branch에 반영한다.
>
> `git push --all` : 모든 로컬 branch를 한번에 push 한다.
>
> `git push --tags` :  기본적인 push는 tag를 자동으로 push 하지않는다. --tag 옵션으로 모든 tag를 push 할 수 있다. Tag에 대해서는 추후 배운다.



## 2.3 버전 관리 및 브랜치

지금까지 쌓아온 버전 이력을 확인하고 버전을 변경하거나 버전 사이의 차이를 확인하는 명령어들과 branch를 이용하는 명령어들을 알아본다.



### git log

> 현재까지 쌓인 commit들을 볼 수 있다. Commit의 hash code, 저자 정보, 날짜 시간, commit 메시지를 보여준다. log 보기 상태에서 'q'를 눌러야 빠져나올 수 있다.
>
> `git log` : 기본
>
> `git log --oneline` : 한 줄에 commit 하나씩 보여주기
>
> `git log --graph` : 변경 이력을 그래프로 보여주기
>
> `git log -p` : 각 commit의 텍스트 변경 사항을 직접 보여주기
>
> `git log -n` : 최근 n개의 commit만 보기
>
> `git log  --since="4 weeks"` : 지난 4주 동안의 로그 보기. 시간 단위는 seconds/minutes/hours/days/weeks/months/years 등 모두 가능
>
> `git log --before="2 days"` : 2일 전까지의 로그 보기

------

### git reset

> 버전 관리에서 매우 중요하면서도 조심히 써야 하는 verb다. 과거 버전으로 돌아가는 명령어다. 돌아갈 특정 commit은 commit hash나 가장 최근 commit을 나타내는 HEAD로 commit을 지정한다. 세 가지 방식이 있는데 유의해서 써야 한다. 자세한 설명은 [이곳]([https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-Reset-%EB%AA%85%ED%99%95%ED%9E%88-%EC%95%8C%EA%B3%A0-%EA%B0%80%EA%B8%B0](https://git-scm.com/book/ko/v2/Git-도구-Reset-명확히-알고-가기))에서 그림을 보는것이 좋다.
>
> `git reset --soft <past_commit>` : 작업 트리의 내용은 바꾸지 않고 HEAD만 "past_commit" 으로 변경한다. 인덱스 공간을 건드리지 않으므로 "past_commit" 이후의 변경 사항은 stage 되어있다. filename을 쓰면 특정 파일만 변경하고 
>
> `git reset --mixed <past_commit>` :  --mixed는 기본 옵션이다. 작업 트리의 내용은 바꾸지 않고 HEAD를 "past_commit" 으로 변경한다. 인덱스 공간을 "past_commit"으로 덮어써서 "past_commit" 이후의 변경된 파일은 Unstaged 상태가 된다.
>
> `git reset --hard <past_commit>` : 작업 트리와 인덱스 공간 모두를 "past_commit" 버전으로 덮어써버린다. 파일의 상태는 Unmodified가 된다.
>
> `git reset <filename>` : 특정 파일에서 stage한 내용을 취소하여 Unstaged (Modified or Untracked) 상태로 만든다. add 명령을 취소한다고 보면 된다.
>
> `git reset --soft <past_commit> <filename>` : 작업 트리의 파일을 변경하지 않고 최근 commit을 취소한다. 실행 후 Staged 상태가 된다.

------

### git diff

> diff는 두 상태의 차이를 보여주는 verb다. 변경 사항을 확인할 때 매우 유용하다.
>
> `git diff` : 작업 트리와 인덱스의 차이점 보기 (Unstaged 변경 사항 보기)
>
> `git diff --cached` : 인덱스와 저장소(HEAD)의 차이점 보기 (Staged 변경 사항 보기)
>
> `git diff HEAD` : 작업 트리와 저장소(HEAD)의 차이점 보기
>
> `git diff <start> [end]` : start로부터의 변경 사항이나 start와 end 사이의 변경 사항을 본다. start와 end는 commit hash나 HEAD~n, branch 명, 태그 명이 될 수 있다.

------

### git merge

> `git merge <other_branch>` : 현재 branch에서 other_branch의 commit들을 병합한다.
>
> `git merge [--ff / --no-ff / --ff-only] <other_branch>` : 병합하는 방식을 지정한다. 기본은 --ff (fast-forward)인데 --no-f (non-fast-forward)와의 차이는 [이곳](<https://backlog.com/git-tutorial/kr/stepup/stepup1_4.html>)에서 보는 것이 좋다. `--only-ff`는 fast-forward 방식이 가능할 때만 merge를 하라는 것이다.
>
> `git merge --squash <other_branch>` : other_branch의 모든 commit들을 하나의 commit으로 합쳐서 병합한다.

------

### git branch

> Git을 이용한 협업의 핵심인 branch를 관리하는 verb다.
>
> `git branch` : 로컬 저장소의 branch 목록을 보여준다.
>
> `git branch -r` : 원격 저장소의 branch 목록을 보여준다.
>
> `git branch <branch_name>` : 새로운 branch를 만든다. 
>
> `git branch -m <old_name> <new_name>` : branch의 이름을 변경한다.
>
> `git branch -d <branch_name>` : branch를 삭제한다. HEAD에 병합되지 않은 branch를 삭제하려면 (즉 branch의 commit을 영구적으로 삭제하려면) -D 옵션을 준다.

------

### git checkout

> HEAD를 다른 commit 혹은 branch로 옮기고 작업 트리를 그 commit의 snapshot으로 복원한다. 목적지를 다른 branch로 지정하면 그 branch의 최신 commit으로 HEAD가 옮겨지고 작업 트리가 바뀐다. 그래서 주로 작업 branch를 변경하는데 주로 쓰인다.
>
> `git checkout <other_branch>` : other_branch로 작업 branch를 바꾸고 작업 트리 영역을 other_branch의 최신 commit 상태로 복원한다.
>
> `git checkout -b <new_branch>` : 현재 상태에서 새로운 branch를 생성하고 그곳으로 branch를 옮긴다. HEAD의 commit이 변하지 않고 단지 branch만 바뀐다. 그래서 작업 트리도 변하지 않는다.
>
> `git checkout HEAD -- <filename>` : 파일의 상태를 HEAD (최신 commit)으로 복원하는 명령어다. 잘못된 변경 사항이 있을 때 주로 쓴다.

------



# 참고자료



[https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EC%99%80-Merge-%EC%9D%98-%EA%B8%B0%EC%B4%88](https://git-scm.com/book/ko/v2/Git-브랜치-브랜치와-Merge-의-기초)

[https://medium.com/@joongwon/git-git-%EB%AA%85%EB%A0%B9%EC%96%B4-%EC%A0%95%EB%A6%AC-c25b421ecdbd](https://medium.com/@joongwon/git-git-명령어-정리-c25b421ecdbd)

<https://blog.outsider.ne.kr/572>

<https://brainbackdoor.tistory.com/15>

