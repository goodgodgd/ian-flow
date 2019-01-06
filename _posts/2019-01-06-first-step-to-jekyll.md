---
layout: post
title:  "지킬(Jekyll)을 이용한 블로그 만들기"
date:   2019-01-06 09:00:00
categories: jekyll
---

# 지킬(Jekyll)을 이용한 블로그 만들기

### 들어가기 전에

같은 제목으로 구글을 검색하면 비슷한 내용을 담고 있는 블로그들이 많이 나온다. 아마 다들 지킬을 처음 적용하기까지 노력이 많이 들었기에 그 과정을 기록해두려 한 것 같다. 나 또한 내가 아는 범위에서 나에게 가장 편했던 방법을 기록하고자 한다. 여기서는 **윈도우**에서 **GitLab Page**에 블로그를 작성하는 방법을 정리한다.

### 지킬이란 무엇인가? 

답은 이 [블로그]에서 잘 설명한 것 같다. 내용을 좀 따오면
> Jekyll 은 아주 심플하고 블로그 지향적인 정적 사이트 생성기입니다. Jekyll 은 다양한 포맷의 원본 텍스트 파일을 템플릿 디렉토리로부터 읽어서, (Markdown 등의) 변환기와 Liquid 렌더러를 통해 가공하여, 당신이 즐겨 사용하는 웹 서버에 곧바로 게시할 수 있는, 완성된 정적 웹사이트를 만들어냅니다. 그리고 Jekyll 은 GitHub Pages 의 내부 엔진이기도 합니다. 다시 말해, Jekyll 을 사용하면 자신의 프로젝트 페이지나 블로그, 웹사이트를 무료로 GitHub 에 호스팅 할 수 있다는 뜻입니다.
> 

github 등에서 사용하는 마크다운을 정적 사이트로 변환해 주는 것은 그냥 github에서 readme 보는 거랑 비슷하지만 다양한 테마를 적용하면 멋진 블로그를 만들 수 있다.  

### 왜 지킬인가?

블로그 플랫폼은 네이버, 티스토리, 이글루 등 많이 있지만 아래와 같은 지킬의 장점을 만족하지는 못 한다.
- 마크다운을 이용하여 텍스트로 모든 효과를 간단하게 넣을 수 있다. (블로그 작성 생산성이 높다)
- git을 통해 버전 관리를 할 수 있다.
- MathJax라는 플러그인을 통해 Tex으로 **수식을 예쁘게 넣을 수 있다.** (제일 중요!)
- 마크다운에 코드 삽입시 자동으로 **문법 하일라이팅**이 된다. (중요2)
- github이나 gitlab에서 무료로 블로그를 올릴 수 있다.

한마디로 공돌이에 최적화된 블로그다. 블로그를 만들고 관리하기 위해서는 어느 정도 코딩과 git에 대한 지식이 필요하므로 진입장벽이 좀 있다고 할 수 있다.

지킬은 정적 사이트 생성기 이므로 마크다운으로 포스트 작성 후 직접 지킬로 html을 생산할 수 있다. 하지만 보통 쓰는 방법은 GitHub page 등의 지킬을 지원하는 무료 웹 호스팅 서비스를 사용하는 것이다. 나는 **FREE private** repository를 쓰기 위해 **GitLab**을 사용하므로 뒤 내용은 GitLab을 기반으로 작성한다.

### 그래서 뭐부터 하면 되지?

많은 지킬 사용법을 설명하는 블로그들이 다짜고짜 **루비(Ruby)**부터 설치하라고 하는데 그 이유는 지킬 자체가 루비 패키지(정확히 말하면 gem)이기 때문이다. 루비는 프로그래밍 언어인데 루비 자체가 필요하다기 보다는 그와 함께 설치되는 패키지 관리 프로그램인 `bundler`가 필요하다. 일단 [여기](https://rubyinstaller.org/downloads/)서 루비를 다운로드 받으면서 다음 글을 읽자. 반드시 **WITH DEVKIT**을 설치해야 한다. 나는 현재 버전 2.4.5를 사용중인데 일단은 최신 버전을 깔아보고 안되는게 있으면 이전 버전을 설치해보자.

**그러나**  

꼭 블로그를 만들기 위해 지킬을 PC에 설치해야 하는것은 아니다. GitHub이나 GitLab 같은 경우 웹에서 바로 마크다운 작성이 가능하기 때문에 간단한 포스트는 웹에서 작성하여 확인할 수도 있다. 하지만 마크다운이 업데이트 된 후 실제 블로그에 반영되기까지는 시간이 좀 걸린다. 그래서 수식이나 코드 이미지 등이 많이 들어간 복잡한 포스트를 작성하며 자주 수정하며 확인하기에는 적절하지 않다. 그래서 로컬에 설치된 지킬을 통해 미리 정적 사이트를 생성하며 잘 작성된 것을 확인 한 후 GitLab으로 push 하여 작성하는 것이 일반적이다. push 한 후 GitLab에서 하는 일도 똑같이 지킬을 이용해 블로그의 html을 생성하는 것이다.

### GitHub 저장소 만들기

[GitHub](https://github.com/)에 들어가서 계정을 만들고 아래 그림처럼 나오면 `New Repository`를 누르고 이름만 정해주면 된다. 프로젝트 명을 `blog` 라고 지으면 블로그 url이 `[username]/github.io` 로 나오고 다르게 지으면 `[username]/github.io/[project name]` 으로 나온다. 나같은 경우는 `https://goodgodgd.github.io/ian-flow/`가 되었다.
![github new project](/ian-flow/assets/2019-01-06-first-step-to-jekyll/github-new-project.png)

### 제일 어려운 건? 테마 선택!

당신이 html과 css에 능하고 시간이 좀 여유있으며 원하는 블로그의 이미지가 확고하다면 바닥부터 블로그 테마를 만들어도 되겠으나 나같이 웹 개발자도 아닌 C++과 파이썬 밖에 모르는 사람은 테마를 골라쓰는 게 옳다! http://jekyllthemes.org/ 에서는 일일이 다 볼수도 없을 만큼 다양한 블로그 테마를 제공한다. 문제는 너무 많아서 다 보기도 어려운데 딱 맘에 드는건 별로 없다는 것이다:)

내가 고른것은 별로 예쁘지도 않은 [EasyBook](http://laobubu.net/jekyll-theme-EasyBook/) 테마다. 현재 욕심으로는 많은 글을 올리고 싶은데 생각보다 카테고리를 보여주는 블로그 테마가 별로 없다. 그정도는 알아서 다 만들수 있을 거라고 생각하는 건지... 어쨌건 카테고리도 있고 별로 멋은 없지만 깔끔한것 같아 이 테마를 선택했다. 테마를 선택했으면 테마 파일을 다운로드 받자. 

### 테마 수정

테마 파일을 받고 나서 일단 `.gitlab-ci.yml`부터 만들자. GitLab에서 지킬을 실행하는 설정파일이다. [여기](https://gitlab.com/pages/jekyll#gitlab-ci)를 참조하여 다음과 같이 만든다.

```yml
image: ruby:2.5

variables:
  JEKYLL_ENV: production

pages:
  before_script:
  - apt-get -y update > /dev/null
  - apt-get install -y locales
  - echo "en_US UTF-8" > /etc/locale.gen
  - locale-gen en_US.UTF-8
  - export LANG=en_US.UTF-8
  - export LANGUAGE=en_US:en
  - export LC_ALL=en_US.UTF-8
  
  script:
  - bundle install
  - bundle exec jekyll build -d public
  artifacts:
    paths:
    - public
  only:
  - master
```

`image`에서는 설치한 루비 버전을 적고 `only` 밑에 있는 것은 블로그로 출력할 브랜치 이름이다. 윈도우에서 사용하면서 인코딩 문제가 좀 있는데 나같은 경우 `before_script` 단락을 추가하여 해결하였다. [여기](https://gitlab.com/pages/jekyll#gitlab-ci)에 나온 yml 파일과 비교해보면 차이를 알 것이다.

지킬에서 가장 중요한 것은 `_config.yml` 파일이다. 테마마다 config 파일의 구성은 좀 다른데 테마에 있는 블로그 제목이나 사용자명, 이메일 등을 자신의 것으로 바꾸면 된다.

### 간단한 포스트 작성하고 로컬에서 확인

`_post` 폴더 아래 `YYYY-MM-DD-post-name.md` 형식으로 마크다운 파일을 만들어 보자. 내용은 간단하게 적어도 된다. `README.md` 에 쓰이는 마크다운과의 차이점은 다음과 같은 FrontMatter가 필요하다는 것이다.

```
---
layout: post
title: "first post"
date: 2019-01-06 09:00:00
categories: mycategory
---
```

`layout`은 바꿀 필요가 없고 `title, date, categories` 등을 원하는 대로 수정하면 된다. FrontMatter는 반드시 가장 위에 있어야 한다. 포스트를 확인했으면 지킬로 웹페이지를 만들어보자.

윈도우에서 루비를 설치했다면 `시작`에 `Start command prompt with Ruby`가 있을 것이다. 그걸 열고 디렉토리를 `cd /path/to/theme`으로 이동해보자. 먼저 필요한 `gem`들을 설치한다. 이를 위해서 반드시 `Gemfile` 파일이 있는 디렉토리 위치에서 명령을 실행한다. 혹시 `Gemfile.lock`이 있다면 지우고 실행한다.

```
$ gem install jekyll bundler
$ bundle
```

이후 아래와 같은 명령을 치고 서버가 돌아가는지 확인하자.

```
$ bundle exec jekyll serve
D:\Work\jekyll\ian-flow>bundle exec jekyll serve
Configuration file: D:/Work/jekyll/ian-flow/_config.yml
            Source: D:/Work/jekyll/ian-flow
       Destination: D:/Work/jekyll/ian-flow/_site
 Incremental build: disabled. Enable with --incremental
      Generating...

Filename                                              | Count |  Bytes |  Time
------------------------------------------------------+-------+--------+------
_layouts/default.html                                 |     7 | 45.95K | 0.050
_includes/head.html                                   |     7 |  7.17K | 0.021
...
중략
...

                    done in 1.901 seconds.
  Please add the following to your Gemfile to avoid polling for changes:
    gem 'wdm', '>= 0.1.0' if Gem.win_platform?
 Auto-regeneration: enabled for 'D:/Work/jekyll/ian-flow'
    Server address: http://127.0.0.1:4000/ian-flow/
  Server running... press ctrl-c to stop.
```

명령창에 보이는 `server address` (http://127.0.0.1:4000/ian-flow/)를 복사해서 인터넷 주소창에 붙여넣어보자. 블로그가 보인다면 성공이다. 자신이 만든 포스트도 어떻게 나오는지 확인해보자.

### GitLab Page에서 확인




