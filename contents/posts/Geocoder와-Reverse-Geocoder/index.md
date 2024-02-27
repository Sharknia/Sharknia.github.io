---
IDX: "NUM-147"
tags:
  - ETC
  - Python
description: "Geocoder에 대한 조사"
update: "2024-02-27T05:52:00.000Z"
date: "2024-02-27"
상태: "Ready"
title: "Geocoder와 Reverse Geocoder"
---
![](image1.png)
## Geocoder와 Reverse Geocoder란? 

간단하게 이야기해 주소로 위도와 경도를 얻거나(Geocoder) 위도와 경도로 주소를 얻는(Reverse Geocoder) 것을 의미합니다. 

### 왜 필요할까? 

주소로는 서로간의 거리를 계산하기 힘들기 떄문에 변환이 필요하고, 위도와 경도는 사람이 읽기 힘들기 때문에 다시 주소로 변환이 필요합니다. 이 때 사용하는 기술입니다. 

## geopy

파이썬에서 geopy를 사용하면 간단하게 Reverse Geocoder와 Geocoder를 구현해볼 수 있습니다. 

테스트에 앞서 우선 geopy를 설치해줍니다.

```bash
pip install geopy
```

### Geocoder 예제

```bash
from geopy.geocoders import Nominatim

# Nominatim 서비스 초기화 (user_agent 설정 필요)
geolocator = Nominatim(user_agent="geoapiExercises")

# 변환하고자 하는 주소
address = "서울특별시 중구 세종대로 110"

# 주소를 위도와 경도로 변환
location = geolocator.geocode(address)

if location:
    print((location.latitude, location.longitude))
else:
    print("좌표를 찾을 수 없습니다.")

```

위의 코드를 실행하면 해당 주소에 해당하는 좌표(위도와 경도) 값을 얻을 수 있습니다. 

### Reverse Geocoder 예제

```python
from geopy.geocoders import Nominatim

geolocator = Nominatim(user_agent="geoapiExercises")
# 예시 좌표, language 인자를 reverse 메소드 호출 시 사용
location = geolocator.reverse("37.541578, 126.840436", exactly_one=True, language="ko")

if location:
    print(location.address)
else:
    print("주소를 찾을 수 없습니다. ")

```

변환된 주소를 얻을 수 있습니다. 

```bash
강서로, 화곡1동, 강서구, 서울특별시, 07726, 대한민국
```

reerse method에 language를 함께 넣어주면 한글 주소를 얻을 수 있습니다. 

### 이외에는?

`geopy`는 지오코딩과 역 지오코딩 외에도 다양한 기능을 제공합니다:

1. 거리 계산: `geopy.distance`를 사용하면 두 지점(위도와 경도 좌표) 사이의 거리를 계산할 수 있습니다. 여러 가지 거리 계산 방법(예: Vincenty, Great Circle)을 제공합니다.

1. 다양한 지오코더 지원: `Nominatim` 외에도 `GoogleV3`, `Bing`, `OpenCage`, `ArcGIS` 등 다양한 지오코딩 서비스를 지원합니다. 각 서비스마다 특징과 사용 방법이 다를 수 있습니다.

1. 유연한 API 사용: `geopy`는 사용자가 다양한 외부 지오코딩 서비스에 쉽게 접근할 수 있도록 돕습니다. 이를 통해 개발자는 특정 서비스의 API 제한이나 사용 제한에 구애받지 않고 서비스를 선택할 수 있습니다.

### Nominatim?

위의 예시에서는 `Nominatim` 을 사용했습니다. `Nominatim`은 OpenStreetMap 데이터를 기반으로 하는 지리적 위치 조회 서비스로, 공개된 무료 API 이지만 공정한 사용 정책이 걸려있어 실제 서비스에서 사용하기에는 무리가 있습니다. 

## 그 밖에 다른 서비스는? 

그렇다면 상용화를 위해서는 어떤 서비스를 이용해야 할까요? 간략하게 찾아서 정리해봤습니다. 국내에서는 크게 네이버/카카오/구글을 사용할 수 있습니다. 

### Naver Map API

- 월 300만건 무료, REST API 지원, 300만개 초과시 건당 0.5원

- [https://www.ncloud.com/product/applicationService/maps](https://www.ncloud.com/product/applicationService/maps)

- [https://guide.ncloud-docs.com/docs/maps-reversegeocoding-api](https://guide.ncloud-docs.com/docs/maps-reversegeocoding-api)

### Kakao Map API

- 월 300만건, 하루 30만건, 초과 시 협의 필요

- [https://developers.kakao.com/docs/latest/ko/getting-started/quota](https://developers.kakao.com/docs/latest/ko/getting-started/quota)

- [https://apis.map.kakao.com/web/documentation/#services\_Geocoder](https://apis.map.kakao.com/web/documentation/#services_Geocoder)

### Google Maps Platform

- 10만회 이하 : 1000회당 5달러

- 

- [월간 200달러의 무료 크레딧을 제공합니다.](https://mapsplatform.google.com/pricing/?hl=ko) 

[https://mapsplatform.google.com/pricing/?hl=ko](https://mapsplatform.google.com/pricing/?hl=ko)

[https://developers.google.com/maps/documentation/geocoding/usage-and-billing?hl=ko](https://developers.google.com/maps/documentation/geocoding/usage-and-billing?hl=ko)

10만회 ~ 50만회 : 1000회당 4달러

50만회 이상 : 협의



