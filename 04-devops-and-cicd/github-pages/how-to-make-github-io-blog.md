[gitHub & git](#github--git)   
[- 1. abarthdew.github.io 리포지토리 생성](#1-abarthdewgithubio-리포지토리-생성)   
[- 2. 리포지토리에 index.html 생성](#2-리포지토리에-indexhtml-생성)   
[- 3. git bash로 리포지토리 clone](#3-git-bash로-리포지토리-clone)   
[👁️‍🗨️ 시행착오 과정 기록](#👁️‍🗨️-시행착오-과정-기록)   
[Ruby & Jekyll](#ruby--jekyll)   
[- 1. Jekyll 테마 적용을 위한 Ruby 설치](#1-jekyll-테마-적용을-위한-ruby-설치)   
[- 2. Jekyll 설치](#2-jekyll-설치)   
[- 3. 테마 적용](#3-테마-적용)   
[Theme 메뉴얼 참고](#theme-메뉴얼-참고)   
[gh-pages 브랜치가 죽어도 안 만들어짐](#gh-pages-브랜치가-죽어도-안-만들어짐)   
[- 댓글 기능 추가](#댓글-기능-추가)   
[- 다른 정보 변경](#다른-정보-변경)   
[참고자료](#참고자료)   

# gitHub & git

## 1. abarthdew.github.io 리포지토리 생성

## 2. 리포지토리에 index.html 생성

- [abarthdew.github.io](http://abarthdew.github.io) 주소로 들어가보면 index.html 내 문구가 떠 있는 것을 확인할 수 있음

## 3. git bash로 리포지토리 clone

### 👁️‍🗨️ 시행착오 과정 기록

- 경로 파악
- [https://coding-factory.tistory.com/749](https://coding-factory.tistory.com/749)

```bash
kbizcowk40@FC-KBIZ043 MINGW64 /
$ cd

kbizcowk40@FC-KBIZ043 MINGW64 ~
$ ls
 AppData/
'Application Data'@
 Contacts/
 Cookies@
 Desktop/
 Documents/
 Downloads/
 Favorites/
 Links/
'Local Settings'@
 Music/
'My Documents'@
 NTUSER.DAT
 NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TM.blf
 NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000001.regtrans-ms
 NTUSER.DAT{016888bd-6c6f-11de-8d1d-001e0bcde3ec}.TMContainer00000000000000000002.regtrans-ms
 NetHood@
 Pictures/
 PrintHood@
 Recent@
'Saved Games'/
 Searches/
 SendTo@
 Templates@
 Videos/
 ntuser.dat.LOG1
 ntuser.dat.LOG2
 ntuser.ini
 ntuser.pol
'시작 메뉴'@
```

- 특정 폴더로 이동

```bash
kbizcowk40@FC-KBIZ043 MINGW64 ~
$ cd Downloads/

kbizcowk40@FC-KBIZ043 MINGW64 ~/Downloads
$ ls
PortableGit/                       desktop.ini
PortableGit-2.37.1-64-bit.7z.exe*  gifcam-7-0.zip
VSCodeUserSetup-x64-1.69.2.exe*    jekyll-theme-chirpy-master.zip
WinMerge-2.16.20-x64-Setup.exe*    test/

kbizcowk40@FC-KBIZ043 MINGW64 ~/Downloads
$ cd test
```

- 계정 보안 관련 오류

```bash
kbizcowk40@FC-KBIZ043 MINGW64 ~/Downloads/test
$ git clone https://github.com/ABarthDew/abarthdew.github.io.git
Cloning into 'abarthdew.github.io'...
fatal: unable to access 'https://github.com/ABarthDew/abarthdew.github.io.git/': SSL certificate problem: self signed certificate in certificate chain
```

- 계정 등록 및 sslVerify 해제

```bash
kbizcowk40@FC-KBIZ043 MINGW64 ~/Downloads/test
$ git config --global user.name "ABarthDew"

kbizcowk40@FC-KBIZ043 MINGW64 ~/Downloads/test
$ git config --global user.email "___@gmail.com"

kbizcowk40@FC-KBIZ043 MINGW64 ~/Downloads/test
$ git config --global http.sslVerify false
```

- 완료

```bash
kbizcowk40@FC-KBIZ043 MINGW64 ~/Downloads/test
$ git clone https://github.com/ABarthDew/abarthdew.github.io.git
Cloning into 'abarthdew.github.io'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (6/6), done.
```

- fatal: credential-cache unavailable; no unix socket support 안 나오게 하기

```jsx
auswo@DESKTOP-54PLDV3 MINGW64 ~/Downloads/PortableGit/this-is-Java (main)
$ git pull origin main
fatal: credential-cache unavailable; no unix socket support
fatal: credential-cache unavailable; no unix socket support
From https://github.com/abarthdew/this-is-Java
 * branch            main       -> FETCH_HEAD
Already up to date.

auswo@DESKTOP-54PLDV3 MINGW64 ~/Downloads/PortableGit/this-is-Java (main)
$ git config --system --list
core.symlinks=false
core.autocrlf=true
core.fscache=true
color.diff=auto
color.status=auto
color.branch=auto
color.interactive=true
help.format=html
diff.astextplain.textconv=astextplain
rebase.autosquash=true
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
**credential.helper=!"C:/Users/auswo/Downloads/PortableGit/mingw64/libexec/git-core/git-credential-cache.exe"**

auswo@DESKTOP-54PLDV3 MINGW64 ~/Downloads/PortableGit/this-is-Java (main)
$ git config --system --unset credential.helper

auswo@DESKTOP-54PLDV3 MINGW64 ~/Downloads/PortableGit/this-is-Java (main)
$ git config --system --list
core.symlinks=false
core.autocrlf=true
core.fscache=true
color.diff=auto
color.status=auto
color.branch=auto
color.interactive=true
help.format=html
diff.astextplain.textconv=astextplain
rebase.autosquash=true
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true

auswo@DESKTOP-54PLDV3 MINGW64 ~/Downloads/PortableGit/this-is-Java (main)
$ git pull origin main
From https://github.com/abarthdew/this-is-Java
 * branch            main       -> FETCH_HEAD
Already up to date.
```

# Ruby & Jekyll

## 1. Jekyll 테마 적용을 위한 Ruby 설치

- [https://rubyinstaller.org/downloads/](https://rubyinstaller.org/downloads/)
- 호환성 문제로 2.7.6 버전 설치함
- 1, 2, 3 + Enter

![Untitled](https://raw.githubusercontent.com/abarthdew/abarthdew.github.io/master/assets/img/2022/1%281%29.png)

## 2. Jekyll 설치

```bash
ruby -v // ruby 버전 확인
gem install jekyll bundler // jekyll 설치
```

## 3. 테마 적용

- local 리포지토리에 테마 내용 복사 + 붙여넣기
    ![http://jekyllthemes.org/themes/jekyll-theme-chirpy/](https://raw.githubusercontent.com/abarthdew/abarthdew.github.io/master/assets/img/2022/1%282%29.png)
    _http://jekyllthemes.org/themes/jekyll-theme-chirpy/_
    
- 테마의 gem 파일 설치

```bash
C:\Users\auswo\Downloads\test\abarthdew.github.io>gem install bundler
Successfully installed bundler-2.3.19
Parsing documentation for bundler-2.3.19
Done installing documentation for bundler after 0 seconds
1 gem installed
```

- 관련 번들 설치

```bash
C:\Users\auswo\Downloads\test\abarthdew.github.io>bundler install
Fetching gem metadata from https://rubygems.org/..........
Resolving dependencies...
Using public_suffix 4.0.7
Using bundler 2.3.19
Using colorator 1.1.0
Using concurrent-ruby 1.1.10
Using eventmachine 1.2.7 (x64-mingw32)
Using http_parser.rb 0.8.0
Using ffi 1.15.5 (x64-mingw32)
Using forwardable-extended 2.6.0
Using mercenary 0.4.0
Using rb-fsevent 0.11.1
Using liquid 4.0.3
Using rouge 3.30.0
Using safe_yaml 1.0.5
Using unicode-display_width 1.8.0
Using addressable 2.8.0
Using em-websocket 0.5.3
Fetching racc 1.6.0
Fetching rainbow 3.1.1
Fetching parallel 1.22.1
Fetching yell 2.2.2
Fetching rexml 3.2.5
Fetching jekyll-paginate 1.1.0
Fetching thread_safe 0.3.6
Using i18n 1.12.0
Using sassc 2.4.0 (x64-mingw32)
Fetching wdm 0.1.1
Fetching webrick 1.7.0
Using rb-inotify 0.10.1
Using pathutil 0.16.2
Fetching ethon 0.15.0
Using terminal-table 2.0.0
Using jekyll-sass-converter 2.2.0
Using listen 3.7.1
Using jekyll-watch 2.2.1
Installing rainbow 3.1.1
Installing parallel 1.22.1
Installing jekyll-paginate 1.1.0
Installing yell 2.2.2
Installing racc 1.6.0 with native extensions
Installing rexml 3.2.5
Installing webrick 1.7.0
Installing wdm 0.1.1 with native extensions
Installing ethon 0.15.0
Installing thread_safe 0.3.6
Fetching tzinfo 1.2.10
Installing tzinfo 1.2.10
Using kramdown 2.4.0
Using kramdown-parser-gfm 1.1.0
Using jekyll 4.2.2
Fetching jekyll-redirect-from 0.16.0
Fetching jekyll-archives 2.2.1
Fetching jekyll-sitemap 1.4.0
Fetching jekyll-seo-tag 2.8.0
Installing jekyll-archives 2.2.1
Installing jekyll-redirect-from 0.16.0
Installing jekyll-seo-tag 2.8.0
Installing jekyll-sitemap 1.4.0
Fetching typhoeus 1.4.0
Installing typhoeus 1.4.0
Using jekyll-theme-chirpy 5.2.1 from source at `.`
Fetching tzinfo-data 1.2022.1
Installing tzinfo-data 1.2022.1
Fetching nokogiri 1.13.8 (x64-mingw32)
Installing nokogiri 1.13.8 (x64-mingw32)
Fetching html-proofer 3.19.4
Installing html-proofer 3.19.4
Bundle complete! 6 Gemfile dependencies, 47 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

- 리포지토리 내 _config.yml 파일 수정(url, username 등)
- git push

```bash
git add *
git commit -m "commit"
git push origin main
```

- 결과 : 오류

# Theme 메뉴얼 참고

- [https://chirpy.cotes.page/posts/getting-started/](https://chirpy.cotes.page/posts/getting-started/)
- [https://www.irgroup.org/posts/jekyll-chirpy/](https://www.irgroup.org/posts/jekyll-chirpy/)
- git bash로 아래 명령

```bash
auswo@DESKTOP-LBOABJ3 MINGW64 ~/Downloads/test/abarthdew.github.io (main)
$ bash tools/init.sh
warning: LF will be replaced by CRLF in .gitignore.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in _config.yml.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in .github/workflows/pages-deploy.yml.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in Gemfile.lock.
The file will have its original line endings in your working directory
[INFO] Initialization successful!
```

- 의존성 설치

```bash
auswo@DESKTOP-LBOABJ3 MINGW64 ~/Downloads/test/abarthdew.github.io (main)
$ bundle
Using public_suffix 4.0.7
Using addressable 2.8.0
Using bundler 2.3.19
Using colorator 1.1.0
Using concurrent-ruby 1.1.10
Using eventmachine 1.2.7 (x64-mingw32)
Using http_parser.rb 0.8.0
Using em-websocket 0.5.3
Using ffi 1.15.5 (x64-mingw32)
Using ethon 0.15.0
Using forwardable-extended 2.6.0
Using mercenary 0.4.0
Using racc 1.6.0
Using nokogiri 1.13.8 (x64-mingw32)
Using parallel 1.22.1
Using rainbow 3.1.1
Using typhoeus 1.4.0
Using yell 2.2.2
Using html-proofer 3.19.4
Using i18n 1.12.0
Using sassc 2.4.0 (x64-mingw32)
Using jekyll-sass-converter 2.2.0
Using rb-fsevent 0.11.1
Using rb-inotify 0.10.1
Using listen 3.7.1
Using jekyll-watch 2.2.1
Using rexml 3.2.5
Using kramdown 2.4.0
Using kramdown-parser-gfm 1.1.0
Using liquid 4.0.3
Using pathutil 0.16.2
Using rouge 3.30.0
Using safe_yaml 1.0.5
Using unicode-display_width 1.8.0
Using terminal-table 2.0.0
Using jekyll 4.2.2
Using jekyll-archives 2.2.1
Using jekyll-paginate 1.1.0
Using jekyll-redirect-from 0.16.0
Using jekyll-seo-tag 2.8.0
Using jekyll-sitemap 1.4.0
Using jekyll-theme-chirpy 5.2.1 from source at `.`
Using thread_safe 0.3.6
Using tzinfo 1.2.10
Using tzinfo-data 1.2022.1
Using wdm 0.1.1
Using webrick 1.7.0
Bundle complete! 6 Gemfile dependencies, 47 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
1 installed gem you directly depend on is looking for funding.
  Run `bundle fund` for details
```

- 배포

```bash
$ bundle exec jekyll s(serve)
```

# gh-pages 브랜치가 죽어도 안 만들어짐

- 왜일까 메뉴얼대로 했는데…
- 같은 과정을 계속 반복하다가 GitHub Action 기능을 사용함

![Untitled](https://raw.githubusercontent.com/abarthdew/abarthdew.github.io/master/assets/img/2022/1%283%29.png)

- jekyll configure 사용 버튼을 누르면 pages.yml 파일이 새로 생김 ⇒ 배포 후 정상적으로 gh-pages 브랜치, 블로그 생성 완료

![Untitled](https://raw.githubusercontent.com/abarthdew/abarthdew.github.io/master/assets/img/2022/1%284%29.png)

- 이후 master 브랜치 _post 디렉토리 내 형식에 맞춰 포스트를 쓸 수 있음

## 댓글 기능 추가

- 참고: [https://www.irgroup.org/posts/utternace-comments-system/](https://www.irgroup.org/posts/utternace-comments-system/)

## 다른 정보 변경

- 참고: [https://j1mmyson.github.io/posts/StartingBlog/](https://j1mmyson.github.io/posts/StartingBlog/)
- [https://blog.jaeyoon.io/2017/12/jekyll-image.html](https://blog.jaeyoon.io/2017/12/jekyll-image.html)

# 참고자료

- [https://4d-why.tistory.com/57](https://4d-why.tistory.com/57)
- [https://chirpy.cotes.page/posts/getting-started/](https://chirpy.cotes.page/posts/getting-started/)
- [https://www.irgroup.org/posts/jekyll-chirpy/](https://www.irgroup.org/posts/jekyll-chirpy/)
- [https://stackedit.io/](https://stackedit.io/)
