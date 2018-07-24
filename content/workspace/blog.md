+++
title  = "blog"
toc    = true
weight = 2
+++

## Intro
Blogging 환경 구성을 위한 정리.

- Hosting: GitHub Pages
- Site Generator: Hugo
- Theme: Learn
- Commenting: Disqus
- Analytics: Google Analytics
- Editor: Prose.io
- CI: Travis CI
- Google Adsence

## GitHub Pages

[GitHub Pages](https://pages.github.com/)을 통해 GitHub에 static site대한 hosting을 받을 수 있다.

User Page를 만드려면 간단히 "${USER_NAME}.github.io"로 repository를 만들고 여기에 static web resource를 담으면 된다.

## Hugo

[Jekyll](https://jekyllrb.com/)을 많이 쓰는데 [Hugo](https://gohugo.io/)가 redering 속도가 빠르고 보다 간편한것 같다.

#### Install

arch에선 User Repository형태로 제공된다.

```bash
$ yaourt -Sy hugo
```

#### Generate Static Web Pages

새로 site를 만들때 file layout을 만들어준다.

```bash
$ hugo new site ${SITE_NAME}
$ ls ${SITE_NAME}
archetypes  content  data  layouts  static  themes  config.toml
```

## Learn

[Learn](https://matcornic.github.io/hugo-learn-doc/basics/what-is-this-hugo-theme/)이 깔끔하게 쓰기 좋아보여 사용한다.

_config.toml_
```bash
theme = "hugo-theme-learn"
```

## Disqus

[Disqus](https://disqus.com/)를 통해 static site에 comment 기능을 넣을 수 있다.

_config.toml_
```bash
disqusShortname = "${DISQUS_SHORT_NAME}"
```

_layouts/partials/disqus.html_
```
<div id="disqus_thread"></div>
<script type="text/javascript">
(function() {
    // Don't ever inject Disqus on localhost--it creates unwanted
    // discussions from 'localhost:1313' on your Disqus account...
    if (window.location.hostname == "localhost")
        return;
    var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
    var disqus_shortname = '{{ .Site.DisqusShortname }}';
    dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
    (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
<a href="http://disqus.com/" class="dsq-brlink">comments powered by <span class="logo-disqus">Disqus</span></a>
```

위에서 생성한 partial을 single page에서 참조하도록 해준다.
_layouts/_default/single.html_
```
...
{{ partial "disqus.html" . }}
...
```

## Google Analytics

[Google Analytics](https://www.google.com/analytics/)를 통해 Site 유입 추이 등을 확인할 수 있다.

_config.toml_
```bash
...
googleAnalytics = "${GOOGLE_ANALYTICS}"
...
```

[hugo](https://gohugo.io)에서 [internal templates](https://gohugo.io/templates/internal/)을 지원하는 경우 이를 사용하는게 편하다.

_layouts/_default/single.html_
```
...
{{ partial "_internal/google_analytics_async.html" . }}
...
```

## Prose.io
[Prose.io](http://prose.io/)를 통해 markdown editor를 붙여줄 수 있다.
github url을 직접 써도 괜찮지만, 조금 더 나은 작성 환경을 제공 해줄 수 있다.

_config.toml_
```toml
...
[params]
  editURL = "http://prose.io/#keyolk/keyolk.github.io/edit/hugo/content/"
...
```

위 경로는 아래와 같이 page header로 들어가게 된다.

```bash
$ grep -nri editurl themes/hugo-theme-learn/layouts/partials/
layouts/partials/header.html:46:                {{ if and (or .IsPage .IsSection) .Site.Params.editURL }}
layouts/partials/header.html:51:                    <a class="github-link" href="{{ $Site.Params.editURL }}{{ replace $File.Dir "\\" "/" }}{{ $File.LogicalName }}" target="blank">
```

## Travis CI
[Jekyll](https://jekyllrb.com/)과는 달리 [Hugo](https://gohugo.io/)는 [GitHub Pages](https://pages.github.com/)에서 직접 site generation을 지원해주지 않는다.
[Travis CI](travis-ci.org)를 통해 commit이 등록될때 마다 site를 생성, 배포되게 할 수 있다.

_.travis.yml_
```yaml
language: go

go:
  - "1.8.3"

env:
  global:
    - GIT_NAME="Chanhun Jeong"
    - GIT_EMAIL="keyolk@gmail.com"
    - SOURCE_DIR="public"
    - BUILD_BRANCH="hugo"
    - DEPLOY_BRANCH="master"

install:
  - wget https://github.com/gohugoio/hugo/releases/download/v0.43/hugo_0.43_Linux-64bit.deb
  - sudo dpkg -i hugo*.deb

script:
  - hugo

deploy:
  - provider: pages
    github_token: ${GITHUB_TOKEN}
    local_dir: public
    skip_cleanup: true
    target_branch: master
    on:
      branch: hugo
```

## Google Adsense
광고를 붙이고자할땐 [Google Adsence](https://www.google.com/adsense)가 제일 편하다.

사용하고 있는 
[Learn](https://matcornic.github.io/hugo-learn-doc/basics/what-is-this-hugo-theme/) Theme엔 custom-header를 정의할 수 있는데,

[관련 guide](https://support.google.com/adsense/answer/181947?hl=en&ref_topic=28893&visit_id=1-636680046495836692-3681375313&rd=1)를 참고해서 script가
single page에 들어 갈 수 있게 만들어준다.

```bash
$ find themes/ | grep custom-header
themes/hugo-theme-learn/layouts/partials/custom-header.html
```

_layouts/partials/cutom-header.html_
```html
<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<script>
  (adsbygoogle = window.adsbygoogle || []).push({
    google_ad_client: "ca-pub-5222442308579087",
    enable_page_level_ads: true
  });
</script>
```

## Lunr.js
검색 기능을 제공하려 할때 사용할 수 있다.

사용하고 있는 
[Learn](https://matcornic.github.io/hugo-learn-doc/basics/what-is-this-hugo-theme/) Theme에서 기본적으로 포함되어 있지만. 한국어 indexing을 지원하지 않는다.

아래에서 forking된 lunr.js를 사용하면 한글도 indexing 되어 검색할 수 있다.
https://github.com/codepiano/lunr.js

theme의 lunr.js를 안쓰도록 아래와 같이 theme 내 해당 script의 경로를 찾아서

```bash
$ find ./themes | grep lunr
./themes/hugo-theme-learn/static/js/lunr.min.js
```

같은 경로에 수정된 lunr.js를 둔다.
```
$ mkdir -p ./static/js
$ wget https://raw.githubusercontent.com/codepiano/lunr.js/master/lunr.min.js ./static/js/
```
