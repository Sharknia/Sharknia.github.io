---
tags:
  - ASP.Net
update: "2024-02-01"
date: "2023-08-22"
상태: "Ready"
title: "Nuget 패키지 dll 추출"
---
1. `.nupkg` 파일의 확장자를 `.zip`으로 변경합니다. 예를 들어, `tiktokensharp.1.0.6.nupkg`을 `tiktokensharp.1.0.6.zip`으로 변경합니다.

1. ZIP 압축 해제 도구 (예: WinRAR, 7-Zip 등)를 사용하여 변경된 `.zip` 파일을 엽니다.

1. `.zip` 파일 내에서 `lib` 폴더나 해당하는 폴더를 찾아 `.dll` 파일을 찾습니다.

1. 해당 `.dll` 파일을 추출합니다.

이렇게 하면 `.nupkg`에서 원하는 `.dll` 파일을 얻을 수 있습니다.



이렇게 하면 의존성, 버전 호환성 등의 문제가 발생할 수 있고 NuGet 패키지 관리 기능도 사용할 수 없기 때문에 권장하지 않는 방법입니다.

