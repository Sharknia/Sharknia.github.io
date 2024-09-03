---
IDX: "NUM-192"
tags:
  - Go
  - OAuth2
description: "Go를 활용한 OAuth 2.0 구현"
update: "2024-09-03T09:01:00.000Z"
date: "2024-05-28"
상태: "Ready"
title: "Go를 활용한 OAuth 2.0 구현"
---
![](image1.png)
## 필요 라이브러리 설치

[Go 프로젝트를 생성](https://sharknia.github.io/Go-입문)한 후 OAuth 인증을 위한 라이브러리를 설치해야 합니다. 

```bash
$ go get -u -v github.com/go-oauth2/oauth2/v4/...
```

## Server.go 작성

### 기본

```go
package main

import (
	"log"
	"net/http"

	"github.com/go-oauth2/oauth2/v4/errors"
	"github.com/go-oauth2/oauth2/v4/manage"
	"github.com/go-oauth2/oauth2/v4/models"
	"github.com/go-oauth2/oauth2/v4/server"
	"github.com/go-oauth2/oauth2/v4/store"
)

func main() {
	manager := manage.NewDefaultManager()
	// token memory store
	manager.MustTokenStorage(store.NewMemoryTokenStore())

	// client memory store
	clientStore := store.NewClientStore()
	clientStore.Set("000000", &models.Client{
		ID:     "000000",
		Secret: "999999",
		Domain: "http://localhost",
	})
	manager.MapClientStorage(clientStore)

	srv := server.NewDefaultServer(manager)
	srv.SetAllowGetAccessRequest(true)
	srv.SetClientInfoHandler(server.ClientFormHandler)

	srv.UserAuthorizationHandler = func(w http.ResponseWriter, r *http.Request) (userID string, err error) {
		return "000000", nil
	}

	srv.SetInternalErrorHandler(func(err error) (re *errors.Response) {
		log.Println("Internal Error:", err.Error())
		return
	})

	srv.SetResponseErrorHandler(func(re *errors.Response) {
		log.Println("Response Error:", re.Error.Error())
	})

	http.HandleFunc("/authorize", func(w http.ResponseWriter, r *http.Request) {
		err := srv.HandleAuthorizeRequest(w, r)
		if err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
		}
	})

	http.HandleFunc("/token", func(w http.ResponseWriter, r *http.Request) {
		srv.HandleTokenRequest(w, r)
	})

	log.Fatal(http.ListenAndServe(":9096", nil))
}
```

그리고 나서 해당 파일을 run 합니다. 

```bash
$ go run server.go 
```

이 이후에 [http://localhost:9096/token?grant\_type=client\_credentials&client\_id=000000&client\_secret=999999&scope=read](http://localhost:9096/token?grant_type=client_credentials&client_id=000000&client_secret=999999&scope=read)로 접속하면 token을 발급 받을 수 있습니다. 

### 심화 - JWT 토큰 생성 추가 

Server.go를 다음과 같이 수정합니다.

```go
package main

import (
	"log"
	"net/http"

	"github.com/go-oauth2/oauth2/v4/errors"
	"github.com/go-oauth2/oauth2/v4/generates"
	"github.com/go-oauth2/oauth2/v4/manage"
	"github.com/go-oauth2/oauth2/v4/models"
	"github.com/go-oauth2/oauth2/v4/server"
	"github.com/go-oauth2/oauth2/v4/store"
	"github.com/golang-jwt/jwt"
)

func main() {
	manager := manage.NewDefaultManager()
	// token memory store
	manager.MustTokenStorage(store.NewMemoryTokenStore())

	// client memory store
	clientStore := store.NewClientStore()
	clientStore.Set("000000", &models.Client{
		ID:     "000000",
		Secret: "999999",
		Domain: "http://localhost",
	})
	manager.MapClientStorage(clientStore)

	srv := server.NewDefaultServer(manager)
	srv.SetAllowGetAccessRequest(true)
	srv.SetClientInfoHandler(server.ClientFormHandler)

	manager.MapAccessGenerate(generates.NewJWTAccessGenerate("", []byte("your_secret_key"), jwt.SigningMethodHS512))

	srv.UserAuthorizationHandler = func(w http.ResponseWriter, r *http.Request) (userID string, err error) {
		return "000000", nil
	}

	srv.SetInternalErrorHandler(func(err error) (re *errors.Response) {
		log.Println("Internal Error:", err.Error())
		return
	})

	srv.SetResponseErrorHandler(func(re *errors.Response) {
		log.Println("Response Error:", re.Error.Error())
	})

	http.HandleFunc("/authorize", func(w http.ResponseWriter, r *http.Request) {
		err := srv.HandleAuthorizeRequest(w, r)
		if err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
		}
	})

	http.HandleFunc("/token", func(w http.ResponseWriter, r *http.Request) {
		srv.HandleTokenRequest(w, r)
	})

	log.Fatal(http.ListenAndServe(":9096", nil))
}
```

MapAccessGenerate 메소드 부분이 추가되었습니다. `your_secret_key`는 jwt 토큰 발급에 사용되는 시크릿 키입니다. HS512 알고리즘을 사용하였습니다. 

이렇게 하면 access token이 HS512 알고리즘을 사용한 jwt 토큰으로 변경됩니다. 

### 심화 2 - JWT 토큰 커스텀

Server.go를 다음과 같이 수정합니다. 

```go
package main

import (
	"context"
	"log"
	"net/http"
	"time"

	"github.com/go-oauth2/oauth2/v4"
	"github.com/go-oauth2/oauth2/v4/errors"
	"github.com/go-oauth2/oauth2/v4/generates"
	"github.com/go-oauth2/oauth2/v4/manage"
	"github.com/go-oauth2/oauth2/v4/models"
	"github.com/go-oauth2/oauth2/v4/server"
	"github.com/go-oauth2/oauth2/v4/store"
	"github.com/golang-jwt/jwt"
)


type CustomJWTAccessGenerate struct {
    generates.JWTAccessGenerate
}

// NewCustomJWTAccessGenerate creates a new CustomJWTAccessGenerate instance
func NewCustomJWTAccessGenerate(secretKey []byte) *CustomJWTAccessGenerate {
    return &CustomJWTAccessGenerate{
        JWTAccessGenerate: generates.JWTAccessGenerate{
			SignedKeyID: "",
            SignedKey: secretKey,
            SignedMethod: jwt.SigningMethodHS512,
        },
    }
}


func (a *CustomJWTAccessGenerate) Token(ctx context.Context, data *oauth2.GenerateBasic, isGenRefresh bool) (string, string, error) {
    now := time.Now()
    exp := now.Add(data.TokenInfo.GetAccessExpiresIn())

    // Create custom claims
    claims := jwt.MapClaims{
        "iss":    "your_service_name",
        "aud":    data.Client.GetID(),
        "exp":    exp.Unix(),
        "iat":    now.Unix(),
        "sub":    data.UserID,
        "custom": "custom_value", // Add custom field here
    }

    token := jwt.NewWithClaims(a.SignedMethod, claims)
    access, err := token.SignedString(a.SignedKey)
    if err != nil {
        return "", "", err
    }

    var refresh string
    if isGenRefresh {
        refresh = data.TokenInfo.GetRefresh()
    }

    return access, refresh, nil
}
func main() {
	manager := manage.NewDefaultManager()
	// token memory store
	manager.MustTokenStorage(store.NewMemoryTokenStore())

	// client memory store
	clientStore := store.NewClientStore()
	clientStore.Set("000000", &models.Client{
		ID:     "000000",
		Secret: "999999",
		Domain: "http://localhost",
	})
	manager.MapClientStorage(clientStore)

	srv := server.NewDefaultServer(manager)
	srv.SetAllowGetAccessRequest(true)
	srv.SetClientInfoHandler(server.ClientFormHandler)

    manager.MapAccessGenerate(NewCustomJWTAccessGenerate([]byte("your_secret_key")))

	srv.UserAuthorizationHandler = func(w http.ResponseWriter, r *http.Request) (userID string, err error) {
		return "000000", nil
	}

	srv.SetInternalErrorHandler(func(err error) (re *errors.Response) {
		log.Println("Internal Error:", err.Error())
		return
	})

	srv.SetResponseErrorHandler(func(re *errors.Response) {
		log.Println("Response Error:", re.Error.Error())
	})

	http.HandleFunc("/authorize", func(w http.ResponseWriter, r *http.Request) {
		err := srv.HandleAuthorizeRequest(w, r)
		if err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
		}
	})

	http.HandleFunc("/token", func(w http.ResponseWriter, r *http.Request) {
		srv.HandleTokenRequest(w, r)
	})

	log.Fatal(http.ListenAndServe(":9096", nil))
}

```

`NewCustomJWTAccessGenerate` 를 선언하고, 해당 구조체를 사용해서 Token을 생성하는 것으로 코드를 수정해주었습니다. 

`Token` 메서드에서 원하는대로 JWT 토큰을 커스텀 할 수 있습니다. 

## 주의

`oauth2` 라이브러리는 [이 라이브러리](https://github.com/golang-jwt/jwt)를 사용합니다. 해당 라이브러리는 최신 버전이 5.x로, 깃허브에서 설치법을 확인할 수 있습니다. 다만 oauth2 라이브러리에서는 3.2.2 버전을 사용하고 있어 저 설치법대로 설치를 한다면 충돌이 일어나 코드가 제대로 작동하지 않습니다. 

oauth2 라이브러리를 설치한다면 3.2.2 버전이 제대로 설치되므로, 추가로 해당 라이브러리를 설치할 필요가 없습니다. 

## 참조

[https://github.com/go-oauth2/oauth2/?tab=readme-ov-file](https://github.com/go-oauth2/oauth2/?tab=readme-ov-file)

