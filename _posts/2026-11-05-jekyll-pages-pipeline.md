---
layout: post
title: "Jekyll 4 + GitHub Actions로 Pages 배포 파이프라인 구성하기"
date: 2026-11-05 21:00:00 +0900
categories: [infra]
tags: [jekyll, github-actions, github-pages, ci-cd]
---

이 블로그는 Jekyll 4로 빌드하고 GitHub Actions로 GitHub Pages에 배포합니다.
구성하면서 내린 결정과, 첫 배포에서 실제로 겪은 실패 두 가지를 정리합니다.

## 왜 `github-pages` gem이 아니라 Actions인가

GitHub Pages의 기본 방식("Deploy from a branch")은 `github-pages` gem으로 빌드합니다.
이 gem은 **Jekyll 3.x에 고정**되어 있고, 허용된 플러그인만 쓸 수 있습니다.

Actions로 직접 빌드하면 빌드 환경을 저장소가 통제합니다.

| 항목 | Deploy from a branch | GitHub Actions |
|---|---|---|
| Jekyll 버전 | 3.x 고정 | `Gemfile.lock`이 결정 (4.4) |
| 플러그인 | 허용 목록만 | 제한 없음 |
| 빌드 로그 | 제한적 | 단계별 전체 로그 |
| 재현성 | GitHub 쪽 환경에 의존 | 로컬·CI가 같은 lockfile 사용 |

## 파이프라인 구조

```text
push (main) ──▶ build job ──────────────────────────▶ deploy job
                 checkout                               actions/deploy-pages
                 ruby/setup-ruby (bundler-cache)        └─ environment: github-pages
                 configure-pages                           (main 브랜치만 허용)
                 jekyll build (JEKYLL_ENV=production)
                 upload-pages-artifact
```

핵심은 **build와 deploy를 job으로 분리**한 것입니다.
build는 `contents: read`만으로 충분하고, 배포 권한(`pages: write`, `id-token: write`)은
deploy 단계에서만 실제로 쓰입니다. 배포는 OIDC 토큰으로 인증하므로 저장소에 별도 시크릿이 필요 없습니다.

## 첫 배포에서 겪은 실패 1: 러너에 없는 Ruby 패치 버전

처음에는 `.ruby-version`(3.3.6)을 그대로 CI에서도 쓰도록 했습니다.

```text
Error: Unavailable version 3.3.6 for ruby on ubuntu-26.04
  Only the latest patch release of each series is built for newer runner images.
```

새 러너 이미지에는 각 시리즈의 **최신 패치만** 제공됩니다.
그래서 CI에서는 시리즈만 지정하도록 바꿨습니다.

```yaml
- uses: ruby/setup-ruby@v1
  with:
    ruby-version: "3.3"   # 항상 최신 3.3.x
    bundler-cache: true
```

로컬·클라우드 개발 환경의 패치 버전과 CI의 패치 버전이 달라지지만,
의존성은 `Gemfile.lock`이 고정하므로 빌드 결과에는 영향이 없습니다.
"런타임은 시리즈로, 의존성은 lockfile로" 고정하는 쪽이 운영 부담이 적습니다.

## 첫 배포에서 겪은 실패 2: 두 배포 경로의 경합

워크플로를 추가한 뒤에도 Pages 설정의 Source가 "Deploy from a branch"로 남아 있으면,
`main`에 push할 때마다 **두 개의 배포가 동시에** 돕니다.

- 우리가 만든 `Deploy Jekyll site to Pages` (Jekyll 4)
- GitHub 내장 `pages build and deployment` (Jekyll 3)

둘 다 성공하고, **나중에 끝난 쪽이 사이트를 덮어씁니다.**
실패가 아니라 성공으로 보이기 때문에 놓치기 쉽습니다.
**Settings → Pages → Source = GitHub Actions**로 바꾸면 내장 배포는 더 이상 돌지 않습니다.

## 정리

- 빌드 환경은 저장소가 소유한다: Actions + `Gemfile.lock`
- 런타임은 시리즈 단위로, 의존성은 lockfile로 고정한다
- 배포 경로는 하나만 둔다: Source를 GitHub Actions로
- build와 deploy를 job으로 나누고, 배포 인증은 OIDC로 처리해 시크릿을 두지 않는다
