LOFFER: LOFTER에서 벗어나는 데 도움을 주는 소프트웨어
LOFFER는 GitHub에 배포할 수 있는 Jekyll 블로그로, 코드 작성이나 커맨드 라인 사용 없이 블로그를 배포할 수 있습니다.

현재 문서와 기본 튜토리얼을 분리하였으며, LOFFER의 기능 및 업데이트 사항을 설명합니다. 코드 지식이 없는 사용자들을 위한 튜토리얼은 여기에서 확인하세요.

업데이트 내용
2019-07-25 V0.4.0
디렉토리 점프 문제 수정 (완벽한 해결은 아니지만 더 이상 문제가 발생하지 않음)
LaTeX 렌더링 지원 추가, 관련 설명 및 예시 참조
고정된 게시글 기능 추가: 게시글의 YAML Front Matter에 pinned: true를 추가하면 해당 글이 고정됩니다.
LOFFER의 테마 색상 변경 방법 소개: LOFFER는 Open Color 색상표를 사용하고 있으며, 이를 통해 색상을 변경할 수 있습니다. 기본 색상은 teal이며, _sass/_variables.scss 파일에서 teal을 원하는 색으로 변경한 후 커밋하면 됩니다.
2019-07-20 V0.3.0
새 버전에서 디렉토리 기능 추가: 게시글의 정보란에 toc: true를 추가하면 디렉토리가 표시됩니다.
jekyll-toc by allejo를 사용하여 디렉토리 생성
주의: 현재 디렉토리는 데스크톱 버전에서만 표시됩니다.
2019-06-30 V0.2.0
스타일 최적화 및 GitHub Issues 기반의 댓글 시스템 Gitalk 추가 (설정 방법 아래 참조)
LOFFER는 블로그 컨테이너로, 게시글이 블로그의 핵심입니다. 게시글을 저장하는 _post 폴더를 통해 Markdown 형식으로 글을 작성합니다.
지원되는 기능
_post 폴더에 Markdown 형식으로 블로그 글 작성
글의 YAML Front Matter에서 저자, 고정된 글, 디렉토리 추가 기능 사용
블로그 아카이브 및 태그로 게시글 관리
소셜 미디어 링크 연결 (_config.yml에서 설정)
Disqus와 Gitalk 댓글 시스템 지원 (_config.yml에서 설정)
예시 YAML 포맷
---
layout: post
title: Markdown 문법 소개
date: 2013-07-16
Author: Shengbin
tags: [sample, markdown]
comments: true
toc: true
---
기여 방법
이 프로젝트를 다른 사람들에게 소개하여 LOFTER에서 벗어날 수 있도록 도와주세요.

문제나 Pull Request는 언제든지 환영합니다.

별 하나 주시면 감사하겠습니다!

img

감사의 말씀
Jekyll - LOFFER의 기본 플랫폼
Kiko-now - 처음에는 이 테마를 포크하고 수정하여 LOFFER가 탄생했습니다.
Font Awesome - 소셜 네트워크 아이콘은 FontAwesome의 무료 오픈 소스 콘텐츠입니다.
