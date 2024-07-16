---
layout: single
title: "github 페이지 만들다 - with Jekyll"
date: 2024-07-09 17:47:41 +0900
categories: jekyll github
---

#### 요약
Github Pages 를 만들었던 기록. 대략적인 작업 순서와 설치하면서 겪었던 문제 등을 기록한다.

#### 작업 순서

1. github 에 새로운 repository 생성하기(참고: [GitHub Pages 사이트 만들기][how to create github pages])
2. Jekyll 사용하기 위해 ruby, gem 설치(참고: [Jekyll on macOS][Jekyll on macOS])  

   > #### Troubleshooting  
   > 1. 이 단계에서 chruby(?) 설치 실패했으나 Jekyll 설치에 문제 없었음!
   > 2. 버전이 안맞는 문제가 있어서 jekyll 버전을 3.6 으로 고정해서 설치했음  
     `sudo gem install jekyll -v 3.6`
3. 로컬에 repository 받고 Jekyll 설치
   ```bash
   $ cd REPOSITORY-DIR
   $ git checkout --orphan gh-pages
   $ git rm -rf .
   $ jekyll new --skip-bundle .
   ```
   설치 후 생성되는 파일들  
   ![설치되면 생성되는 파일들]({{site.url}}/src/imgs/sample_jekyll_1.png){:class="img-responsive"}
4. Gemfile 수정하기  
   ```shell
   # If you want to use GitHub Pages, remove the "gem "jekyll"" above and
   # uncomment the line below. To upgrade, run `bundle update github-pages`.
   gem "github-pages", "~> 231", group: :jekyll_plugins
   ```
   github-pages 버전(`"~> 231"`)은 [Dependency versions][github-versions] 를 참고해서 작성했다.
5. bundle install 실행
   ```bash
   $ bundle install
   ```
6. .gitignore 에 Gemfile.lock 추가하기
7. _config.yml 에서 필요한 설정 수정하기
8. 로컬에서 테스트해보기
   ```bash
   $ jekyll serve
   ```
   실행 후 `http://localhost:4000` 에서 확인할 수 있다.

9. git 에 올리기  
   git 에 올리고 반영되기까지 1~2분 정도 걸린다.

#### 결론
github 에서 새로운 repository 를 생성해서 Jekyll 설치하고 이 포스트 작성까지 2시간 정도 걸렸다. github 의 가이드 잘 따라가니 쉽게 설치 가능했다. 이게 어떤 원리로 동작되는지는 아직 모르겠지만 앞으로 꾸준히 해보면서 알아봐야겠다. Jekyll 에서 다양한 테마를 제공하는 듯 하니 다음번에는 테마 변경을 시도해봐야겠다. 

[Jekyll on macOS]: https://jekyllrb.com/docs/installation/macos/
[how to create github pages]: https://docs.github.com/ko/pages/getting-started-with-github-pages/creating-a-github-pages-site
[github-versions]: https://pages.github.com/versions/