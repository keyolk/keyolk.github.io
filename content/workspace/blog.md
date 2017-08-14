+++
title  = "blog"
toc    = true
weight = 2
prev   = "/workspace/office"
next   = "/workspace"
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
## Google Analytics

[Google Analytics](https://www.google.com/analytics/)를 통해 Site 유입 추이 등을 확인할 수 있다.

_config.toml_
```bash
googleAnalytics = "${GOOGLE_ANALYTICS}"
```

## Prose.io

[Prose.io](http://prose.io/)

## Travis CI

[Travis CI](travis-ci.org)

## Google Adsence

## Troubleshooting

