---
tags:
  - Blogging
  - Hobby
update: "2024-02-01"
date: "2023-08-20"
상태: "Ready"
title: "MORETHAN-LOG 수정"
---
아주 마음에 드는 오픈소스로, 다만 내 취향에 맞게 짜잘짜잘 임의로 몇 가지 부분을 수정했다. 

1. Tags 정렬

    등록한 태그들이 블로그 좌측에 나열되는데, 이를 이름 순서대로 나오게 정렬했다. 

    ```typescript
    // itemObj를 item name으로 정렬
      const sortedItemObj = Object.entries(itemObj)
        .sort((a, b) => a[0].localeCompare(b[0]))
        .reduce((acc, [key, val]) => {
          acc[key] = val
          return acc
        }, {} as { [itemName: string]: number })
    
      return sortedItemObj
    ```

    

1. 자동썸네일 기능 테스트

    src\routes\Detail\PostDetail\PostHeader.tsx 에서 썸네일을 가져오고 있고, 

    src\pages\[slug].tsx 에서 ogImageGenerateURL을 이용한 부분이 있다. 

    다만 ogImageGenerateURL을 이용한 부분은 meta에만 적용되고 있고, 자동 썸네일 용 [https://og-image-korean.vercel.app](https://og-image-korean.vercel.app/) 는 만약 사용을 위해서라면 내 이미지를 활용한것으로 수정해야 할 것 같다. 

