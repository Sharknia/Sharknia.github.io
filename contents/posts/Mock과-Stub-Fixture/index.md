---
IDX: "NUM-202"
tags:
  - TDD
  - Django
  - Python
description: "Mock과 Stub, Fixture"
update: "2024-09-03T08:49:00.000Z"
date: "2024-06-20"
상태: "Ready"
title: "Mock과 Stub, Fixture"
---
## Fixture

**Fixture**는 테스트 실행 전에 필요한 상태나 객체를 설정하는 데 사용됩니다. 주로 데이터베이스 초기화, 파일 시스템 설정, 특정 객체 생성 등의 작업을 포함합니다.

#### 예제

```python
@pytest.fixture
def user(db):
    return User.objects.create_user(
        email="user@example.com",
        dob="1990-01-01",
        gender=1,
        nickname="testnickname",
    )
```

위의 예제는 Django 테스트 환경에서 실제로 `User` 객체를 데이터베이스에 생성합니다. 이 fixture는 테스트 함수가 실행될 때마다 호출되며, 테스트가 끝나면 데이터베이스 상태를 롤백합니다.

## Mock

**Mock**은 테스트 중에 외부 의존성을 대체하는 객체입니다. 실제 객체의 동작을 모방하지만, 더 가볍고 빠르며 예측 가능한 결과를 반환합니다. Mock 객체는 호출된 메서드와 그 인자를 기록할 수 있어서, 테스트 중에 호출 여부와 호출된 인자를 검증할 수 있습니다.

#### 예제

```python
from unittest.mock import MagicMock, patch

def test_s3_upload():
    mock_s3_client = MagicMock()
    mock_s3_client.upload_file.return_value = "mocked_url"

    with patch('common.S3Client', return_value=mock_s3_client):
        s3_client = S3Client()
        result = s3_client.upload_file("fake_path")
        assert result == "mocked_url"
```

위의 예제에서는 `S3Client`의 `upload_file` 메서드를 모킹하여 실제로 파일을 업로드하지 않고 가짜 URL을 반환하도록 합니다.

## Stub

**Stub**은 테스트 중에 의존성을 대체하는 간단한 구현체입니다. 주로 특정한 메서드 호출에 대해 미리 정의된 값을 반환하도록 설정됩니다. Mock과 비슷하지만, 호출된 메서드와 인자를 기록하지 않으며, 더 단순한 형태입니다.

#### 예제

```python
class S3ClientStub:
    def upload_file(self, file_path):
        return "stubbed_url"

def test_s3_upload():
    s3_client = S3ClientStub()
    result = s3_client.upload_file("fake_path")
    assert result == "stubbed_url"
```

위의 예제에서는 `S3ClientStub`을 사용하여 `upload_file` 메서드를 간단하게 스텁(stub) 처리했습니다.

## 차이점 정리

1. **Fixture**: 테스트를 위해 필요한 상태나 객체를 설정합니다. 실제 객체나 데이터베이스 상태를 사용합니다.

1. **Mock**: 외부 의존성을 대체하여 실제 객체 대신 가짜 객체를 사용합니다. 호출된 메서드와 인자를 기록할 수 있습니다.

1. **Stub**: 특정한 메서드 호출에 대해 미리 정의된 값을 반환하는 간단한 구현체입니다. Mock보다 더 단순한 형태입니다.



