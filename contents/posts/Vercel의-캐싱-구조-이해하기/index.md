---
IDX: "NUM-279"
tags:
  - Vercel
  - BackEnd
description: "sitemap을 생성하다 알게된 Vercel의 캐싱 구조"
update: "2026-01-09T04:24:00.000Z"
date: "2026-01-09"
상태: "Ready"
title: "Vercel의 캐싱 구조 이해하기"
---
![](image1.png)
## Vercel에서 사이트맵을 만들다가 배운 것: 커스텀 도메인을 쓸 거라면 환경변수는 필수다

최근 Notion 기반 블로그를 구축하고 있습니다. 구글 서치 콘솔에도 등록을 할 것이기 때문에 SEO를 위한 동적 사이트맵을 구현했습니다. 

`sitemap.ts` 파일 하나 만들면 끝날 줄 알았는데 Vercel의 캐싱 구조를 제대로 이해하지 못해서 삽질을 좀 했습니다.

### 첫 번째 접근: 동적으로 도메인 가져오기

사이트맵은 절대 경로를 요구합니다. `/blog/my-post`가 아니라 `https://example.com/blog/my-post` 형태여야 합니다.

```typescript
// sitemap.ts - 첫 번째 시도
import { headers } from 'next/headers';

export default async function sitemap() {
  const host = headers().get('host');
  const baseUrl = `https://${host}`;

  return posts.map(post => ({
    url: `${baseUrl}/blog/${post.id}`,
  }));
}
```

처음 생각은 이랬습니다: "어차피 Google Search Console에 `blog.tuum.day`(커스텀 도메인)으로 등록할 거고 구글 봇이 그 도메인으로 요청할 테니까 사이트맵용 도메인을 `headers()`로 동적으로 가져오면 되지 않나?"

기능적으로는 맞는 말입니다. 하지만 성능과 비용을 고려하면 이야기가 달라집니다.

### 동적 렌더링의 대가

Next.js App Router에서 `headers()`를 사용하면 해당 라우트는 자동으로 Dynamic Rendering으로 전환됩니다. 즉 캐싱이 안 됩니다.

제 블로그는 Notion API에서 데이터를 가져옵니다. 매 요청마다 API를 호출하면,

- 응답 속도 저하: Notion API는 느립니다

- Rate Limit 리스크: [Notion 공식 문서](https://developers.notion.com/reference/request-limits)에 따르면 평균 3 req/sec 제한이 있습니다

- 불필요한 API 호출: 사이트맵은 블로그 포스트가 추가될 때만 바뀌는데, 매번 새로 만들 이유가 없습니다

그래서 ISR(Incremental Static Regeneration)을 적용하기로 했습니다. ISR은 정적 페이지를 미리 생성해두고, 설정한 시간이 지나면 백그라운드에서 자동으로 갱신하는 Next.js의 캐싱 전략입니다. 1시간에 한 번만 사이트맵을 갱신하면 충분합니다.

<hr style="border: none; height: 1px; background-color: #e0e0e0; margin: 16px 0;" />
### ISR + 동적 도메인 = 문제

"그러면 `headers()`로 도메인 가져오면서 ISR도 적용하면 되지 않나?"

여기서 Vercel의 중요한 특성을 알게 됐습니다.

#### Vercel의 ISR 캐시는 '배포' 단위

Vercel에서 하나의 배포(Deployment)에는 여러 도메인이 연결될 수 있습니다:

- 자동 생성 URL: `my-project-abc123.vercel.app`

- Preview URL: `my-project-git-feature-xyz.vercel.app`

- Custom Domain: `tuum.tech`

저는 이들이 각각 독립적인 캐시를 가질 거라고 생각했습니다. 하지만 [Vercel 공식 문서](https://vercel.com/docs/concepts/incremental-static-regeneration)를 보면 ISR 캐시는 도메인이 아닌 배포를 기준으로 저장됩니다. 캐시 키는 `/sitemap.xml`처럼 경로만 포함합니다.

[object Promise]그리고 "Revalidating across domains"라는 별도 가이드가 있는 것 자체가 도메인 간 캐시가 공유된다는 방증이기도 합니다.

#### Cache Poisoning 시나리오

물론 Preview 배포는 기본적으로 [Vercel Authentication](https://vercel.com/docs/security/deployment-protection)으로 보호됩니다. 로그인 없이는 접근할 수 없습니다.

하지만 Production 배포의 경우 커스텀 도메인과 `*.vercel.app` 도메인이 동시에 공개될 수 있습니다. 이 경우에,

1. ISR 캐시가 만료됨

1. `my-project.vercel.app/sitemap.xml`로 요청이 먼저 들어옴

1. 사이트맵이 `https://my-project.vercel.app/blog/...` 형태로 생성되어 캐싱됨

1. 이후 `tuum.tech/sitemap.xml` 요청 → 잘못된 도메인이 담긴 캐시가 반환됨

커스텀 도메인으로 SEO를 하고 싶은데, 사이트맵에는 `vercel.app` 도메인이 들어가는 상황이 발생할 수 있습니다.

### 해결책: 환경변수로 Base URL 고정

결국 선택한 방법은 단순합니다. 환경변수로 Base URL을 명시하는 것입니다.

```typescript
// sitemap.ts
export const revalidate = 3600; // 1시간 ISR

export default async function sitemap() {
  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || '<https://tuum.tech>';
  const posts = await postRepository.getAllPublishedPaths();

  return [
    { url: baseUrl, lastModified: new Date() },
    ...posts.map(post => ({
      url: `${baseUrl}/blog/${post.id}`,
      lastModified: new Date(post.lastModified),
    })),
  ];
}
```

```bash
# Vercel Dashboard 또는 .env.local
NEXT_PUBLIC_BASE_URL=https://tuum.tech
```

설정할 게 하나 늘어나서 정말 하고 싶지 않았지만 커스텀 도메인을 쓸 거라면 정당한 Trade-off입니다.

<hr style="border: none; height: 1px; background-color: #e0e0e0; margin: 16px 0;" />
### 다른 플랫폼과의 비교

이 특성은 Vercel의 설계 철학입니다. 모든 플랫폼이 이렇지는 않습니다.

| 플랫폼 | 캐시 키에 도메인 포함 여부 |
| --- | --- |
| Vercel | ❌ 배포 단위 (경로만 키) |
| Cloudflare | ⭕ 기본 포함,  | [Custom Cache Key](https://developers.cloudflare.com/cache/concepts/cache-keys/) | 로 제어 가능 |
| AWS CloudFront | ⭕ 캐시 정책에서 Host 헤더 포함 여부 선택 가능 |

Vercel의 접근법은 대부분의 경우 효율적입니다. 같은 배포를 가리키는 도메인들은 보통 동일한 콘텐츠를 서빙하기 때문입니다. 하지만 도메인에 따라 다른 콘텐츠를 생성하거나 특정 커스텀 도메인만 SEO 타겟으로 삼고 싶다면 주의가 필요합니다.

<hr style="border: none; height: 1px; background-color: #e0e0e0; margin: 16px 0;" />
### 정리

| 방식 | 캐싱 | 도메인 제어 | 권장 상황 |
| --- | --- | --- | --- |
| `headers()` |  동적 | ❌ | ⭕ | 트래픽 적고 API 부담 없을 때 |
| 환경변수 + ISR | ⭕ | ⭕ | 커스텀 도메인으로 SEO할 때 |
| `headers()` |  + ISR | ⭕ | ❌ | 사용 금지 |

인프라의 캐싱 동작을 정확히 이해하지 않으면 코드는 완벽해도 시스템은 예상과 다르게 동작합니다.



