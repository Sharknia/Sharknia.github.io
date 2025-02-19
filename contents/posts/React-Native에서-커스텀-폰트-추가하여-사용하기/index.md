---
IDX: "NUM-245"
tags:
  - React Native
  - Typescript
description: "React Native에서 커스텀 폰트 추가하여 사용하기"
update: "2025-02-19T12:44:00.000Z"
date: "2025-02-19"
상태: "Ready"
title: "React Native에서 커스텀 폰트 추가하여 사용하기"
---
![](image1.png)
## 서론

이번에 만들어볼 어플리케이션에서는 폰트를 모두 Noto Sans KR로 통일하려고 합니다. 기복없이 튀지 않으면서 안정적인 폰트 같습니다. 

그래서 리액트 네이티브 프로젝트에서 폰트를 어떻게 적용하면 될지를 기록하겠습니다. 

## **프로젝트 준비**

### 폰트 다운로드

프로젝트 내의 ./assets/fonts/ 폴더에 .ttf 파일들을 넣어둡니다. 폰트는 [이 곳](https://fonts.google.com/noto/specimen/Noto+Sans+KR)에서 다운로드 했습니다. 

### **react-native.config.js 파일**

이 파일은 리액트 네이티브가 추가 Asset(폰트, 이미지 등)의 경로를 인식할 수 있도록 설정하는 파일입니다. 프로젝트 루트에 아래와 같이 생성합니다.

```javascript
module.exports = {
  assets: ['./assets/fonts/'],
};
```

폰트, 이미지, 사운드 파일 등의 정적 파일들을 네이티브 프로젝트(iOS, Android)로 자동 복사하는 경로를 정의합니다. 즉, 이 설정을 통해 빌드 시 폰트가 네이티브 프로젝트에 자동으로 반영되도록 미리 선언하는 느낌이라고 보면 됩니다. 수동으로 react-native link를 실행하지 않고도, 필요한 자산이 빌드 과정에서 자동으로 복사되도록 설정할 수 있게 해 효율을 올려줍니다. 

만약 굳이 수동으로 Asset 파일들을 네이티브 프로젝트(iOS, Android)에 직접 복사하고 설정할 경우, react-native.config.js 파일은 필요 없습니다.

## **npx react-native-asset 명령어**

npx react-native-asset 명령어는 react-native.config.js에 지정한 자산 경로를 기반으로 폰트 파일들을 iOS와 Android 프로젝트에 자동으로 연결(link)해줍니다. 이 명령어를 실행하면 각 플랫폼의 설정 파일에 폰트 경로가 추가되어 빌드 시 폰트를 사용할 수 있게 됩니다.

터미널에서 다음 명령어를 실행합니다.

```bash
npx react-native-asset
```

## **폰트 사용**

폰트 링크가 완료된 후, 스타일 시트에서 fontFamily 속성을 사용해 적용할 수 있습니다. 예를 들어, 아래와 같이 사용합니다.

```typescript
import React from 'react';
import { Text, StyleSheet } from 'react-native';

const App = () => {
  return (
    <Text style={styles.customText}>안녕하세요, Noto Sans KR 폰트입니다.</Text>
  );
};

const styles = StyleSheet.create({
  customText: {
    fontFamily: 'NotoSansKR-Bold', // 폰트 파일 내에 지정된 폰트 이름
    fontSize: 20,
  },
});

export default App;
```

NotoSansKR-Bold은 실제 폰트 파일 내부의 이름과 일치해야 합니다. 

## Typhograph.ts 작성

이 폰트를 기반으로 css의 클래스처럼 미리 Typhography를 정의해두고 사용하려고 합니다. 

다음과 같이 src/styles에 Typhograph.ts를 생성하고  미리 스타일을 정의해주었습니다. 

```typescript
import { StyleSheet } from 'react-native';

export const typography = StyleSheet.create({
    // Headline
    headlineL: { fontSize: 32, fontFamily: 'NotoSansKR-Bold' },
    headlineM: { fontSize: 24, fontFamily: 'NotoSansKR-Bold' },
    headlineS: { fontSize: 20, fontFamily: 'NotoSansKR-Bold' },

    // Title
    titleL: { fontSize: 16, fontFamily: 'NotoSansKR-Bold' },
    titleM: { fontSize: 14, fontFamily: 'NotoSansKR-Bold' },
    titleS: { fontSize: 12, fontFamily: 'NotoSansKR-Bold' },

    // Body
    bodyL: { fontSize: 16, fontFamily: 'NotoSansKR-Regular' },
    bodyM: { fontSize: 14, fontFamily: 'NotoSansKR-Regular' },
    bodyS: { fontSize: 12, fontFamily: 'NotoSansKR-Regular' },

    // Button
    buttonL: { fontSize: 16, fontFamily: 'NotoSansKR-Bold' },
    buttonM: { fontSize: 14, fontFamily: 'NotoSansKR-Regular' },
    buttonS: { fontSize: 12, fontFamily: 'NotoSansKR-Regular' },

    // Caption
    captionL: { fontSize: 11, fontFamily: 'NotoSansKR-Light' },
    captionM: { fontSize: 10, fontFamily: 'NotoSansKR-Light' },
});

```

이는 컴포넌트에서 폰트를 사용할 때 다음과 같이 사용할 수 있습니다. 

```typescript
import { Text, View } from 'react-native';
import { typography } from 'src/styles/typography';

const ExampleScreen = () => {
  return (
    <View>
      <Text style={typography.headlineL}>큰 헤드라인</Text>
      <Text style={typography.bodyM}>본문 텍스트 예시</Text>
      <Text style={typography.captionM}>작은 캡션 텍스트</Text>
    </View>
  );
};

export default ExampleScreen;
```

