---
layout: post
title:  "지킬(Jekyll)을 이용한 블로그 만들기"
date:   2019-01-06 09:00:00
categories: jekyll
---

# 지킬(Jekyll)을 이용한 블로그 만들기

### 들어가기 전에

같은 제목으로 구글링하면 비슷한 내용을 담고 있는 블로그들이 많이 나온다. 아마 다들 지킬을 처음 적용하기까지 노력이 많이 들었기에 그 과정을 기록해두려 한 것 같다. 나 또한 내가 아는 범위에서 나에게 가장 편했던 방법을 기록하고자 한다. 여기서는 **윈도우**에서 **GitHub Page**에 블로그를 작성하는 방법을 정리한다.

### 지킬이란 무엇인가? 

답은 이 [블로그](https://nachwon.github.io/jekyllblog/)에서 잘 설명한 것 같다. 내용을 좀 따오면
> Jekyll 은 아주 심플하고 블로그 지향적인 정적 사이트 생성기입니다. Jekyll 은 다양한 포맷의 원본 텍스트 파일을 템플릿 디렉토리로부터 읽어서, (Markdown 등의) 변환기와 Liquid 렌더러를 통해 가공하여, 당신이 즐겨 사용하는 웹 서버에 곧바로 게시할 수 있는, 완성된 정적 웹사이트를 만들어냅니다. 그리고 Jekyll 은 GitHub Pages 의 내부 엔진이기도 합니다. 다시 말해, Jekyll 을 사용하면 자신의 프로젝트 페이지나 블로그, 웹사이트를 무료로 GitHub 에 호스팅 할 수 있다는 뜻입니다.
> 

github 등에서 사용하는 마크다운을 정적 사이트로 변환해 주는 것은 그냥 github에서 readme 보는 거랑 비슷하지만 다양한 테마를 적용하면 멋진 블로그를 만들 수 있다.  

### 왜 지킬인가?

블로그 플랫폼은 네이버, 티스토리, 이글루 등 많이 있지만 아래와 같은 지킬의 장점을 만족하지는 못 한다.
- 마크다운을 이용하여 텍스트로 모든 효과를 간단하게 넣을 수 있다. (블로그 작성 생산성이 높다)
- git을 통해 버전 관리를 할 수 있다.
- MathJax라는 플러그인을 통해 Tex으로 **수식을 예쁘게 넣을 수 있다.** (제일 중요!)
- 마크다운에 코드 삽입시 자동으로 **문법 하일라이팅**이 된다. (중요2)
- GitHub이나 GitLab에 무료로 블로그를 올릴 수 있다.

한마디로 공돌이에 최적화된 블로그다. 블로그를 만들고 관리하기 위해서는 어느 정도 코딩과 git에 대한 지식이 필요하므로 진입장벽이 좀 있다고 할 수 있다.

지킬은 정적 사이트 생성기 이므로 마크다운으로 포스트 작성 후 직접 지킬로 html을 생산할 수 있다. 하지만 보통 쓰는 방법은 GitHub page 등의 지킬을 지원하는 무료 웹 호스팅 서비스를 사용하는 것이다. 아래 내용도 GitHub Page를 이용한 방법을 정리한다.

### 그래서 뭐부터 하면 되지?

많은 지킬 사용법을 설명하는 블로그들이 다짜고짜 **루비(Ruby)**부터 설치하라고 하는데 그 이유는 지킬 자체가 루비 패키지(정확히 말하면 gem)이기 때문이다. 루비는 프로그래밍 언어인데 루비 자체가 필요하다기 보다는 그와 함께 설치되는 패키지 관리 프로그램인 `bundler`가 필요하다. 일단 [여기](https://rubyinstaller.org/downloads/)서 루비를 다운로드 받으면서 다음 글을 읽자. 반드시 **WITH DEVKIT**을 설치해야 한다. 나는 현재 버전 2.4.5를 사용중인데 일단은 최신 버전을 깔아보고 안되는게 있으면 이전 버전을 설치해보자.

**그러나** <a id="create-repository"></a> 

꼭 블로그를 만들기 위해 지킬을 PC에 설치해야 하는것은 아니다. GitHub이나 GitLab 같은 경우 웹에서 바로 마크다운 작성이 가능하기 때문에 간단한 포스트는 웹에서 작성하여 확인할 수도 있다. 하지만 마크다운이 업데이트 된 후 실제 블로그에 반영되기까지는 시간이 좀 걸린다. 그래서 수식이나 코드 이미지 등이 많이 들어간 복잡한 포스트를 작성하며 자주 수정하며 확인하기에는 적절하지 않다. 그래서 로컬에 설치된 지킬을 통해 미리 정적 사이트를 생성하며 잘 작성된 것을 확인 한 후 GitHub으로 push 하여 확인하는 것이 일반적이다. push 한 후 GitHub에서 하는 일도 똑같이 지킬을 이용해 블로그의 html을 생성하는 것이다.

### GitHub 저장소 만들기 

[GitHub](https://github.com/)에 들어가서 계정에 로그인 하고 아래 그림처럼 나오면 `New Repository`를 누르고 이름만 정해주면 된다. 프로젝트 명을 `blog` 라고 지으면 블로그 url이 `https://[username]/github.io` 로 나오고 다르게 지으면 `https://[username]/github.io/[repo name]` 으로 나온다. 나같은 경우는 `https://goodgodgd.github.io/ian-flow/`가 되었다.
![github new project](/ian-flow/assets/2019-01-06-first-step-to-jekyll/github-new-project.png)

### 제일 어려운 건? 테마 선택!

당신이 html과 css에 능하고 시간이 좀 여유있으며 원하는 블로그의 이미지가 확고하다면 바닥부터 블로그 테마를 만들어도 되겠으나 나같이 웹 개발자도 아닌 C++과 파이썬 밖에 모르는 사람은 테마를 골라쓰는 게 옳다! [http://jekyllthemes.org/](http://jekyllthemes.org/) 에서는 일일이 다 볼수도 없을 만큼 다양한 블로그 테마를 제공한다. 문제는 너무 많아서 다 보기도 어려운데 딱 맘에 드는건 별로 없다는 것이다:)

내가 고른것은 별로 예쁘지도 않은 [EasyBook](http://laobubu.net/jekyll-theme-EasyBook/) 테마다. 현재 욕심으로는 많은 글을 올리고 싶은데 생각보다 카테고리를 보여주는 블로그 테마가 별로 없다. 그정도는 알아서 다 만들수 있을 거라고 생각하는 건지... 어쨌건 카테고리도 있고 별로 멋은 없지만 깔끔한것 같아 이 테마를 선택했다. 테마를 선택했으면 테마 파일을 다운로드 받자. 

테마 파일 중에서 가장 중요한 것은 `_config.yml` 파일이다. 테마마다 config 파일의 구성은 좀 다른데 테마에 있는 블로그 제목이나 사용자명, 이메일 등을 자신의 것으로 바꾸면 된다.

### 간단한 포스트 작성하고 로컬에서 확인

포스트는 `_post` 폴더 아래 새 마크다운 파일을 작성하면 자동으로 블로그에 새 글이 생긴다. 마크다운 이름을 `YYYY-MM-DD-post-name.md` 형식으로  파일을 만들어 보자. 내용은 간단하게 적어도 된다. `README.md` 에 쓰이는 마크다운과의 차이점은 다음과 같은 Front Matter가 필요하다는 것이다.

```
---
layout: post
title: "first post"
date: 2019-01-06 09:00:00
categories: mycategory
---
```

`layout`은 그대로 두고 `title, date, categories` 등을 원하는 대로 수정하면 된다. Front Matter는 반드시 가장 위에 있어야 한다. 

마크다운 문서를 만들 때의 꿀팁은 좋은 편집기를 사용하는 것이다. 많은 편집기들이 있지만 내가 본 중 가장 세련된 편집기를 하나 추천한다.

[https://typora.io/](https://typora.io/)

typora는 마크다운을 워드처럼 [WISYWIG](https://ko.wikipedia.org/wiki/%EC%9C%84%EC%A7%80%EC%9C%84%EA%B7%B8) 로 편집할 수 있게 해준다. 물론 `ctrl+/`을 누르면 코드 모드로 전환도 가능하다. 심지어 tex 수식도 렌더링해서 보여준다. 내부에서 테마도 바꿀수 있고 여러 기능들이 있으니 잘 찾아쓰길 바란다.

지킬을 로컬에서 돌리기 위해 먼저 루비 환경을 만들어야 한다. Python이 익숙한 사람을 위해 비슷한 용어를 연결해보면

- python → ruby: 프로그래밍 언어
- package → gem: 패키지는 패키지인ㄷ... C++의 라이브러리 정도로 보면 되겠다.
- pipenv → bundler: ruby 및 gem 버전 관리자

윈도우에서 루비를 설치했다면 `시작`에 `Start command prompt with Ruby`가 있을 것이다. 그걸 열고 디렉토리를 `cd /path/to/theme`으로 이동하자. 먼저 번들러를 설치하고 번들러를 이용해 테마에 필요한 gem들을 설치할 것이다. 테마에 필요한 gem들은 테마 안에 있는 `Gemfile` 정의 되어 있다. 아래 명령어를 실행해보자. 혹시 `Gemfile.lock`이 있다면 지우고 실행한다.

```
# 번들러 설치
$ gem install jekyll bundler
# 필요한 gem 설치, 반드시 `Gemfile` 파일이 있는 디렉토리 위치에서 명령을 실행
$ bundle
```

포스트를 작성했으면 지킬로 웹페이지를 만들어보자. jekyll을 이용해서 마크다운으로부터 html을 생산하는 과정이다. 만들어진 html들은 `_site` 폴더 아래 생성된다. 아래와 같은 명령을 치고 서버가 돌아가는지 확인하자.

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

### GitHub Page에서 확인

아까 만든 GitHub 저장소를 사용할 차례다. `git`을 설치하지 않았다면 [여기](https://git-scm.com/download/win)서 다운받아 설치하자. 설치 후 윈도우 탐색기에서 테마 폴더로 들어가 빈 곳에 우클릭하면 `Git Bash Here`라는 메뉴가 보일 것이다. 누르면 터미널이 열리고 원격 저장소 주소를 세팅한다.
```bash
git init
# 현재 지정된 원격 저장소 확인
git remote -v
# 아무것도 없다면 저장소 추가
git remote add origin https://github.com/[username]/[repository_name].git
# 이미 origin이 있다면 내 것으로 변경
git remote set-url origin https://github.com/[username]/[repository_name].git
```
GitHub Page를 이용하기 위해서는 출력할 파일들을 `gh-pages`브랜치에 놓아야 한다. 아래 명령으로 브랜치를 생성하고 현재 브랜치를 바꾸자.

```bash
# 브랜치 생성
git checkout -b gh-pages
# 내용 업데이트 후 원격으로 push
git add .
git commit -m 'your commit message'
git push origin gh-pages
```
GitHub에 소스가 업데이트 되었다면 이제 블로그를 확인해보자. [GitHub 저장소 만들기](#create-repository)에서 말한 것 처럼 `https://[username]/github.io`  혹은 `https://[username]/github.io/[repo name]` 주소로 들어가면 생성된 블로그를 볼 수 있다. 한번에 쭉 성공했... 을리는 없지만 암튼 성공했다면 축하하고 실패했다면 구글신에게 물어보자. 화이팅!!


### 2019.3.2 업데이트: github page를 여러개 만들고 싶다면?

용도가 다른 여러개의 블로그를 동시에 운영하고 싶다면 그냥 새로운 github repo를 만들고 테마 복사해서 넣고 push 하면 된다. 문제는 이렇게 했을 때 공포의 `404` 페이지가 뜬다는 것. (Site not found)  
나 역시 기존에 있던 블로그 소스를 몽땅 복사한 다음 `_config.yml`에서 저장소 이름만 바꿔서 올렸는데 `404` 페이지를 만나게 되었다. 왜 안 될까 고민하면서 뒤적거리다가 `_config.yml` 안의 아래 줄이 맘에 걸렸다.  

```
url: "https://[username].github.io/"
baseurl: "/[repo name]"
```

기존 블로그의 `_config.yml` 파일에도 url이 같이 때문에 혹시혹시 url이 겹쳐서 사이트가 생성되지 않은 걸까? 그래서 url 자체에 baseurl까지 붙여서 전체 주소를 써넣었다.  
`url: "https://[username].github.io/[repo name]"`  

그랬더니 이제 페이지가 보인다!! 오늘도 수고했다며 셀프 칭찬을 하고... ~~자야겠다.~~



### 2021.1.3 업데이트: 수식입력시 주의사항

지킬의 왠만한 테마는 다 기본적으로 mathjax라는 수식 렌더링 서비스를 사용하고 있다. Tex형식으로 수식을 입력하면 아래와 같이 아름다운 수식으로 그려서 보내준다.  

```
$f(x)=x^2$
```

$f(x)=x^2$

그런데 2020년 언젠가부터 이 렌더링이 되지 않아서 `$f(x)=x^2$` 이런 문자열이 그대로 보이게 됐다. 왜 그런고 봤더니 mathjax 버전이 2에서 3으로 올라간것 같다. 기존에 수식 렌더링을 위해 썼던 스크립트는 다음과 같다. 맨마지막에 `latest` 버전을 사용하도록 설정이 돼서 작동이 안하는거라고 추측한다.

```html
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    <!-- TeX: { equationNumbers: { autoNumber: "AMS" } }, -->
    tex2jax: {
      inlineMath: [ ['$','$'], ['\(', '\)'] ],
      displayMath: [ ['$$','$$'] ],
      processEscapes: true,
    }
  });
  MathJax.Hub.Register.MessageHook("Math Processing Error",function (message) {
	alert("Math Processing Error: "+message[1]);
	});
  MathJax.Hub.Register.MessageHook("TeX Jax - parse error",function (message) {
	alert("Math Processing Error: "+message[1]);
	});
</script>

<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
```



근데 MathJax 3.x를 사용하는 방법은 검색해도 많이 안나오고 살짝 해봤는데 행렬이 제대로 렌더링이 되지 않고 이를 해결할 방법을 찾기도 좀 귀찮아서 MathJax 2.x를 계속 사용하도록 설정을 바꿨다. 맨 아래줄을 보면 `2.7.2`로 설정했다.

```html
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    <!-- TeX: { equationNumbers: { autoNumber: "AMS" } }, -->
    tex2jax: {
      inlineMath: [ ['$','$'], ['\\(', '\\)'] ],
      processEscapes: true,
    }
  });
</script>
<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.2/MathJax.js?config=TeX-MML-AM_CHTML'></script>
```



그런데도 기존에 잘 되던 Tex가 렌더링이 잘 안되는 케이스가 있었다. 이건 아마 내가 2.x 버전에서도 낮은 버전을 쓰다가 2.7로 올리면서 생긴 현상 같다.

- **`_` 사용**: $X\_{ab}$ 를 만들고 싶다면 `$X_{ab}$`가 아닌 `$X\_{ab}$`로 써야한다. kramdown에서 `_`를 문자 기울임 표시로 인식 하기 때문에 일단 kramdown에서 `_`를 렌더링 하기 위해서는 `\_`을 써야한다.

- **inline matrix**: inline matrix에서 `\\`를 통해 줄내림이 원래는 됐었는데 이제 안된다. inline matrix에서 줄 내림을 위해서는 `\\\\` 이렇게 네 번의 back slash를 써야한다.

  $ \widehat{\xi} \doteq \begin{bmatrix} \widehat{w} & v \\\\ 0 & 0 \end{bmatrix} $

  ```
  $ \widehat{\xi} \doteq \begin{bmatrix} \widehat{w} & v \\\\ 0 & 0 \end{bmatrix} $
  ```

- 위와 같이 설정하면 `$ ... $` 뿐만 아니라 `\\( ... \\)`으로도 블로그에서 inline 렌더링이 된다. 또한 `\\[ ... \\]`으로는 일반 수식이 렌더링된다. 하지만 Typora에서 안되기 때문에 안 쓸것 같다.



#### 참고자료

- <https://math.meta.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference>: 풍부한 Tex 예제가 있어 참고
- <http://docs.mathjax.org/en/latest/input/tex/html.html>: 기본 Tex와 MathJax 문법 차이
- <http://docs.mathjax.org/en/latest/upgrading/v2.html>: MathJax v2 to v3



