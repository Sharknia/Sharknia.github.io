---
IDX: "NUM-231"
tags:
  - React Native
  - Typescript
description: "React Native의 프로젝트 구조와 화면 이동 구현하기"
update: "2025-01-12T13:32:00.000Z"
date: "2025-01-12"
상태: "Ready"
title: "React Native의 프로젝트 구조와 화면 이동 구현하기"
---
![](image1.png)
## 일반적인 React Native 프로젝트의 구조 

```plain text
root/
├── android/                   # Android 네이티브 코드
├── ios/                       # iOS 네이티브 코드
├── src/                       # 주요 소스 코드 디렉토리
│   ├── assets/                # 이미지, 폰트 등 정적 파일
│   ├── components/            # 재사용 가능한 UI 컴포넌트
│   ├── navigation/            # 네비게이션 관련 코드
│   ├── screens/               # 화면(페이지) 컴포넌트
│   │   ├── LoginScreen.tsx    # 로그인 화면
│   │   ├── HomeScreen.tsx     # 홈 화면
│   │   └── ...                # 기타 화면
│   ├── services/              # API 요청 및 백엔드 연동
│   ├── utils/                 # 유틸리티 함수와 헬퍼 코드
│   ├── context/               # 글로벌 상태 관리 (React Context 등)
│   └── App.tsx                # 메인 엔트리 파일
├── package.json               # 프로젝트 정보 및 의존성 관리
├── tsconfig.json              # 타입스크립트 설정 파일
├── .gitignore                 # Git 무시 파일
└── README.md                  # 프로젝트 설명
```

## 각 디렉토리의 역할

### `src/assets/`

- 이미지, 폰트 등 정적 파일을 저장.

- 예: 아이콘, 배경 이미지, 로고 등.

### `src/components/`

- 재사용 가능한 UI 요소를 정의.

- 예: 버튼, 텍스트 입력 필드, 카드, 모달 등.

    ```plain text
    src/components/
    ├── CustomButton.tsx
    ├── InputField.tsx
    └── Modal.tsx
    ```

### `src/navigation/`

- 화면 간의 라우팅과 네비게이션 설정.

- 예: React Navigation을 사용하여 스택, 탭, 드로어 네비게이션 설정.

    ```plain text
    src/navigation/
    ├── AppNavigator.tsx  # 앱 전체 네비게이션
    ├── AuthStack.tsx     # 로그인/회원가입 스택
    └── MainStack.tsx     # 로그인 후 메인 스택
    ```

###   `src/screens/`

- 각 페이지(화면) 컴포넌트를 정의.

- 페이지별로 파일을 생성하고, 필요하면 서브 디렉토리를 사용.

    ```plain text
    src/screens/
    ├── LoginScreen.tsx
    ├── SignupScreen.tsx
    ├── HomeScreen.tsx
    ├── ProfileScreen.tsx
    └── SettingsScreen.tsx
    ```

###   `src/services/`

- API 요청 및 백엔드와의 연동 로직 작성.

- 예: `axios`나 `fetch`를 사용한 API 호출 함수 정의.

    ```plain text
    src/services/
    ├── api.ts           # 공통 API 설정
    ├── authService.ts   # 인증 관련 API
    └── userService.ts   # 사용자 데이터 API
    ```

###   `src/utils/`

- 재사용 가능한 유틸리티 함수, 상수, 헬퍼 코드.

- 예: 날짜 포맷팅, 폼 유효성 검사 함수 등.

    ```plain text
    src/utils/
    ├── dateUtils.ts
    ├── validation.ts
    ├── constants.ts
    └── helpers.ts
    ```

### `src/context/`

- React Context API를 사용해 글로벌 상태를 관리.

- 예: 사용자 인증 상태, 테마 설정 등.

    ```plain text
    src/context/
    ├── AuthContext.tsx
    ├── ThemeContext.tsx
    └── AppProvider.tsx
    ```

### `App.tsx`

- 프로젝트의 메인 엔트리 파일.

- 네비게이션 초기화 및 최상위 컴포넌트 렌더링.

    ```typescript
    import React from 'react';
    import { NavigationContainer } from '@react-navigation/native';
    import AppNavigator from './src/navigation/AppNavigator';
    
    const App: React.FC = () => {
      return (
        <NavigationContainer>
          <AppNavigator />
        </NavigationContainer>
      );
    };
    
    export default App;
    ```

## AppNavigator를 활용한 화면의 추가

App.tsx가 main.py 같은 역할을 하는 파일임을 짐작할 수 있습니다. 여기서 AppNavigator를 사용하고 있는 것도 확인할 수 있습니다. 

이 페이지는 React Navigation을 사용하여 앱의 화면 전환(네비게이션)을 관리하는 역할을 합니다. React Native 앱에서는 여러 화면이 있고, 이러한 화면 간의 이동을 효과적으로 관리하기 위해 네비게이션 라이브러리를 사용하는 것이 일반적입니다.

`AppNavigator`(또는 React Navigation과 같은 네비게이션 시스템)를 사용하지 않으면, 화면 전환 및 뒤로가기 동작을 직접 관리해야 합니다. 이는 상당히 번거롭고, 유지보수성이 떨어질 수 있습니다. React Navigation을 사용하면 뒤로가기를 포함한 화면 전환과 네비게이션 흐름을 자동으로 관리할 수 있어 훨씬 간편합니다.

React Navigation 같은 라이브러리를 사용하지 않으면,

1. 각 화면 전환을 수동으로 처리해야 합니다.

1. 안드로이드의 뒤로가기 버튼(하드웨어 버튼)도 직접 구현해야 합니다.

1. 상태를 유지하기 위한 글로벌 상태 관리나 Context API를 추가적으로 사용해야 할 수 있습니다.

### 설치

```bash
npm install @react-navigation/native
npm install react-native-screens react-native-safe-area-context react-native-gesture-handler react-native-reanimated react-native-vector-icons
npm install @react-navigation/stack
cd ios && pod install && cd ..
```

###  **주요 역할**

1. **스택 네비게이션 구성**

    - `createStackNavigator`를 통해 화면 간의 전환 방식을 스택(Stack) 방식으로 관리.

    - 스택 방식은 사용자가 한 화면에서 다른 화면으로 이동할 때, 이전 화면이 "뒤로가기"로 돌아갈 수 있도록 스택 구조에 저장되는 방식.

1. **화면 정의**

    - `SplashScreen`: 앱이 처음 로드될 때 사용자에게 표시되는 스플래시 화면.

    - `LoginScreen`: 사용자가 로그인할 수 있는 화면.

    - `HomeScreen`: 로그인 후 메인 화면으로 이동.

1. **초기 화면 설정**

    - `initialRouteName="Splash"`는 앱 실행 시 첫 화면을 `SplashScreen`으로 설정.

    - 앱 로드 후, `SplashScreen`에서 로그인 상태를 확인한 뒤 적절한 화면(`LoginScreen` 또는 `HomeScreen`)으로 리디렉션.

1. **헤더 숨기기**

    - `screenOptions={{ headerShown: false }}`를 통해 화면 상단의 기본 헤더를 숨김.

### 코드

/src/navigation 경로에 다음의 파일을 작성해줍니다. 

```typescript
import { createStackNavigator } from '@react-navigation/stack';
import React from 'react';
import HomeScreen from '../screens/HomeScreen';
import LoginScreen from '../screens/LoginScreen/LoginScreen';
import SplashScreen from '../screens/SplashScreen';

const Stack = createStackNavigator();

const AppNavigator: React.FC = () => {
    return (
        <Stack.Navigator initialRouteName="Splash" screenOptions={{ headerShown: false }}>
            <Stack.Screen name="Splash" component={SplashScreen} />
            <Stack.Screen name="Login" component={LoginScreen} />
            <Stack.Screen name="Home" component={HomeScreen} />
        </Stack.Navigator>
    );
};

export default AppNavigator;
```

#### **네비게이션 초기화**

`createStackNavigator`를 통해 스택 네비게이터를 생성합니다.

```typescript
const Stack = createStackNavigator();
```

#### **스택 구성**

`<Stack.Navigator>`에 앱에서 사용할 화면들을 정의

- `name`: 네비게이션 이름. 다른 화면으로 이동할 때 이 이름으로 참조.

- `component`: 연결된 화면 컴포넌트

```typescript
<Stack.Navigator initialRouteName="Splash" screenOptions={{ headerShown: false }}>
    <Stack.Screen name="Splash" component={SplashScreen} />
    <Stack.Screen name="Login" component={LoginScreen} />
    <Stack.Screen name="Home" component={HomeScreen} />
</Stack.Navigator>
```

## useNavigation 훅을 사용한 화면 전환

React Navigation가 설치되었고 AppNavigator에 화면을 정의했다면, `useNavigation` 훅을 사용해 `navigation` 객체를 간단하게 가져올 수 있습니다.

다만 이 전에 타입스크립트의 특징 때문에 경로와 관련된 타입을 명시해야 오류를 방지할 수 있습니다.

### 타입 정의 추가 

React Navigation의 `createStackNavigator`에서 정의한 경로 이름(`Home`, `Login` 등)에 대한 타입을 추가해야 합니다. types/navigation.tsx 파일을 생성하고 다음의 내용을 작성합니다. 이는 선택사항으로 모든 네비게이션 타입 정의를 한 곳에서 관리하기 위함입니다. 

```typescript
import { StackNavigationProp } from '@react-navigation/stack';

type RootStackParamList = {
  Home: undefined; // 'Home' 경로에 매개변수가 없는 경우
  Login: undefined; // 'Login' 경로
};

type NavigationProp = StackNavigationProp<RootStackParamList, 'Login'>;

export default NavigationProp;
```

`RootStackParamList` 정의

- `Home`, `Login`, `Profile` 등 네비게이션 경로의 이름과 매개변수 타입을 정의.

`useNavigation`에 타입 적용

- `useNavigation<StackNavigationProp<RootStackParamList, 'Login'>>()`와 같이 타입 지정.

### `useNavigation`에 타입 적용

`useNavigation` 훅에 방금 정의한 `NavigationProp` 타입을 적용합니다.

```typescript
import { useNavigation } from '@react-navigation/native';
import React from 'react';
import { StyleSheet, View } from 'react-native';
import SocialButton from '../../components/SocialButton';
import NavigationProp from '../../types/navigation'; // 방금 정의한 타입 가져오기

const LoginScreen: React.FC = () => {
    const navigation = useNavigation<NavigationProp>(); // 타입 적용

    const handleGoogleLogin = () => {
        console.log('Google Login Clicked');
        navigation.navigate('Home'); // 네비게이션 수행
    };

    return (
        <View style={styles.container}>
            <SocialButton
                text="Google 계정으로 로그인"
                icon="google"
                color="#F4A261"
                onPress={handleGoogleLogin}
            />
        </View>
    );
};

export default LoginScreen;
```

### `reset` 메서드

이대로 구현하면 로그인 후 홈 화면으로 이동할 때 **뒤로 가기 버튼**을 누르면 로그인 화면으로 돌아갈 수 있습니다. 이는 일반적으로 기대되는 동작이 아니며, 로그인 화면은 **로그인 후** 접근할 수 없도록 하는 것이 적절합니다.

`navigation.reset`을 사용하면 네비게이션 스택을 재설정하여 뒤로가기를 방지할 수 있습니다. 이는 웹의 **리다이렉트**와 유사한 동작을 합니다.

```typescript
const handleGoogleLogin = () => {
    console.log('Google Login Clicked');

    // 네비게이션 스택을 재설정
    navigation.reset({
        index: 0, // 스택의 첫 번째 화면으로 설정
        routes: [{ name: 'Home' }], // 새로 설정할 스택의 화면
    });
};
```

이렇게 구현하면 로그인 화면은 네비게이션 스택에서 제거되며 "뒤로가기" 버튼을 눌러도 로그인 화면으로 돌아갈 수 없습니다.

## 마무리

오늘은 리액트 네이티브를 설치하고, 프로젝트 구조를 뜯어보고 간단한 화면을 만들어 화면을 이동시켜봤습니다. 

다음 시간에는 하단 탭을 구현하고 하단 탭으로 화면 전환을 하는 방법, 왼쪽/오른쪽에서 열리는 메뉴를 추가해보고 역시 이를 통해 화면을 이동해보겠습니다. 



