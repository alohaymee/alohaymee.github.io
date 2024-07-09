---
layout: post
title: "github 페이지 만들다 - with Jekyll"
date: 2024-07-09 17:47:41 +0900
categories: jekyll github
---

#### 작업 순서

1. github 에 새로운 repository 생성하기(참고: [GitHub Pages 사이트 만들기][how to create github pages])
2. Jekyll 사용하기 위해 ruby, gem 설치(참고: [Jekyll on macOS][Jekyll on macOS])

   - 이 단계에서 chruby(?) 설치 실패했으나 Jekyll 설치에 문제 없었음!
   - 버전이 안맞는 문제가 있어서 jekyll 버전을 3.6 으로 고정해서 설치했음  
     `sudo gem install jekyll -v 3.6`

3. 로컬에 repository 받고 Jekyll 설치

```
$ cd REPOSITORY-DIR
$ git checkout --orphan gh-pages
$ git rm -rf .
$ jekyll new --skip-bundle .
```

[Jekyll on macOS]: https://jekyllrb.com/docs/installation/macos/
[how to create github pages]: https://docs.github.com/ko/pages/getting-started-with-github-pages/creating-a-github-pages-site
