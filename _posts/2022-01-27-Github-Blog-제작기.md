---
layout: post
title: Github Blog 제작기
description: 깃헙 블로그 제작 방법
summary: 깃헙 블로그를 만들면서 겪은 오류와 제작 방법
tags: blog
---

대략적으로 설명하면 다음의 과정을 거치면 된다.

- Github에 새 레포 만들기 (레포 이름은 username.github.io)
- git clone으로 로컬에 폴더 생성
- jekyll 설치 후 적용할 테마 붙여넣기
- push해서 적용

보기에 매우 간단해 보이지만, 하다보니 많은 오류를 마주했다...
오류를 해결한 과정을 위주로 써보려고 한다.

<br>

## jekyll 설치

Ruby의 패키지 매니저인 `gem`을 통해 `jekyll`과 `bundler` 설치를 진행한다.

```bash
gem install jekyll bundler
```

이 과정에서 permission error를 마주할 수 있다.

```bash
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /Library/Ruby/Gems/2.3.0 directory.
```

시스템 Ruby를 이용하고 있기 때문에 권한이 없어 gem 설치가 안된 것이다.

이는 `rbenv`를 이용해 문제를 해결할 수 있다. 시스템에서 관리하고 있는 Ruby가 아닌 rbenv를 이용해 설치한 별도의 Ruby를 이용한다. 만약 brew를 설치하지 않았다면 아래 brew 설치를 먼저 읽으면 된다.

```bash
# brew를 통해 rbenv를 설치한다.
brew install rbenv ruby-build

# rbenv가 잘 설치되었는지 확인
rbenv versions

# rbenve로 관리되는 Ruby를 설치
rbenv install -l # 설치할 수 있는 Ruby의 버전 리스트 확인
rbenv install $(버전) # 확인한 버전 중 최신의 버전을 설치

# rbenv로 글로번 버전을 변경
rbenv global $(버전)

# rbenv PATH 추가
# .zshrc에 다음 스크립트 추가
[[ -d ~/.rbenv  ]] && \
  export PATH=${HOME}/.rbenv/bin:${PATH} && \
  eval "$(rbenv init -)"

# .zshrc 적용
source ~/.zshrc
```

다시 jekyll과 bundler 설치를 진행해본다.

```bash
gem install jekyll bundler
```

<br>

## brew 설치

[https://brew.sh/index_ko](https://brew.sh/index_ko) 에 설치 스크립트를 터미널에서 실행하면 된다.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

설치하는 과정에서 없는 유저네임을 읽으려 해서 계속 실패했다. `.zshrc`에서 `USER`를 따로 정의했는데, 그 부분이 겹치게 되면서 오류를 만들어내고 있었다.

`USER`를 삭제하고 설치를 다시 해봤지만 같은 오류를 내뱉어 포기했다가... 터미널을 껐다 다시 켜니 해결이 되었다. `.zshrc` 수정한 것을 적용을 안시켜줘서 생긴 문제였다.

```bash
source ~/.zshrc
```

수정하면 souce 명령어로 적용! 그렇게 험난하게 brew를 설치했다

<br>

## 테마 적용

jekyll 무료 테마는 여러 사이트에서 찾아볼 수 있다. 맘에 드는 테마를 선택해 적용해주면 된다.

- [https://jekyll-themes.com/free/](https://jekyll-themes.com/free/)
- [https://jekyllthemes.io/](https://jekyllthemes.io/)
- [http://jekyllthemes.org/](http://jekyllthemes.org/)
- [https://jamstackthemes.dev/ssg/jekyll/](https://jamstackthemes.dev/ssg/jekyll/)

맘에 드는 테마가 있다면 다운을 받아 로컬 폴더에 붙여넣어주면 된다. 나는 이 테마를 사용했다.

[https://github.com/P0WEX/Gesko](https://github.com/P0WEX/Gesko)

테마의 README를 읽으면 커스텀 하는 방법에 대해 알 수 있다. 읽으면서 원하는 블로그를 만들면 된다.

기본적인 설정은 `_config.yml`을 수정하면 된다.

<br>

## 확인 및 업데이트

```bash
# 로컬에서 확인
bundle exec jekyll serve

# 라이브 서버
# Jekyll 3.7.0 버전부터 사용 가능
bundle exec jekyll serve --livereload
```

원격에 push해 자신의 블로그를 업데이트하면 된다.

```bash
git add . # 전체 파일
git commit -m '커밋 메시지'
git push
```

<br>

## Reference

[https://jojoldu.tistory.com/288](https://jojoldu.tistory.com/288)

[https://zeddios.tistory.com/1222](https://zeddios.tistory.com/1222)
