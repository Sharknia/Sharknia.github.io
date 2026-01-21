---
IDX: "NUM-281"
tags:
  - Hobby
  - Oracle Cloud
  - BackEnd
  - DevOps
description: "무료 ARM A1 인스턴스 (4 OCPU + 24GB RAM) 생성하기"
update: "2026-01-21T02:15:00.000Z"
date: "2026-01-20"
상태: "Ready"
title: "Oracle Cloud ARM 인스턴스 생성 완전 가이드"
---
## 1. Oracle Cloud 계정 가입

[가입 페이지](https://www.oracle.com/cloud/free/)

[object Promise]### 1.1 결제 정보 등록

- 해외 결제 가능한 신용카드/체크카드 필요

- $1 인증 후 환불됨 (일부 카드는 며칠 소요)

### 1.2 가입 완료 확인

- 이메일 인증 완료

- Oracle Cloud Console 접속 가능 확인

## 2. (Option) Pay As You Go 업그레이드

[object Promise]PAYG 계정이 아니어도 무료 ARM A1 인스턴스 (4 OCPU + 24GB RAM)를 받을 수 있습니다. 

다만 무료 계정의 경우 거의 항상 제한에 걸리고, 커뮤니티 등지에 떠도는 흉흉한 썰로는 이유없이 서버의 접속이 제한되거나 심지어는 회수(!) 당할 수도 있다고 합니다. 

저 역시 1core 1gb ram 서버에 대해서 실제로 이유없이 서버가 3일 정도 정지당한 이력이 있습니다. 

계정을 업그레이드하면 훨씬 수월하게 서버를 할당 받을 수 있으며(실제로 저도 업그레이드와 거의 동시에 바로 서버 생성이 됐습니다), 해당 사양의 서버까지는 계정이 업그레이드 되어도 무료입니다. 

### 2.1 업그레이드 페이지 접속

```plain text
Oracle Cloud Console → ☰ (햄버거 메뉴)
→ Billing & Cost Management (청구 및 비용 관리)
→ Upgrade and Manage Payment (지급 업그레이드 및 관리)
```

### 2.2 업그레이드 진행

1. **[Upgrade]** 버튼 클릭

1. 결제 정보 확인 (기존 카드 사용 또는 새 카드 등록)

1. 약관 동의

1. **[Upgrade Account]** 클릭

### 2.3 업그레이드 확인

- 몇 분 ~ 몇 시간 소요 가능 (저는 세시간 정도 소요됐습니다.)

- 이메일로 완료 알림 수신

- Console에서 계정 유형이 "Pay As You Go"로 표시 확인

#### 2.4 인증 금액 (일시적)

- 약 $100 (또는 €93) 정도가 카드에서 홀드될 수 있음

- 1주일 내 자동 환불 (저는 바로 환불처리 됐습니다.) 

## 3. Budget Alert 설정 (유료 계정 전환의 경우 필수)

[object Promise]사용량만큼 과금이 되는데, 해당 서버 사양은 무료가 맞지만 정말 혹시! 과금이 발생할 수 있으니 알람 설정을 해둡니다. 

### 3.1 Budget 생성

```plain text
☰ → Billing & Cost Management → Budgets(예산) → Create Budget(예산 생성)

```

### 3.2 설정값

| 항목 | 값 |
| --- | --- |
| Name | `free-tier-alert` |
| Target Compartment(예산 범위) | root compartment 선택 |
| Budget Amount (예산 금액) | `1` |  (SG) |
| 임계값 유형 | 예산 백분율 - 100% |
| 전자메일 수신자 | 알림 받을 이메일 |

## 4. OCI CLI 설치

한 번에 생성이 되면 좋지만 유료 계정이라고 하더라도 생성에 실패하는 경우가 있습니다. 따라서 스크립트로 자동화를 해두는 것이 편한데 이를 위해 `oci cli`를 설치합니다. 

#### 4.1 macOS 설치 (Homebrew)

```bash
# Homebrew로 설치 (권장)
brew install oci-cli
```

### 4.2 설치 확인

```bash
# 새 터미널 열고
oci --version

```

## 5. CLI 인증 설정

### 5.1 API Key 생성

```plain text
우측 상단 프로필 → 사용자 설정 → 토큰 및 키 → Add API Key
```

### 5.2 키 페어 생성

1. Generate API Key Pair 선택

1. Download Private Key 클릭 → `oci_api_key.pem` 저장

1. Add 클릭

### 5.3 Configuration File 정보 복사

키 추가 후 나타나는 Configuration File Preview 전체 복사

```plain text
[DEFAULT]
user=ocid1.user.oc1..aaaaaaaaxxxxxxx
fingerprint=xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx
tenancy=ocid1.tenancy.oc1..aaaaaaaaxxxxxxx
region=ap-seoul-1
key_file=<path to your private keyfile>
```

### 5.4 CLI 설정

```bash
# 설정 디렉토리 생성
mkdir -p ~/.oci

# Private Key 이동
mv ~/Downloads/oci_api_key.pem ~/.oci/
chmod 600 ~/.oci/oci_api_key.pem

# Config 파일 생성
cat > ~/.oci/config << 'EOF'
[DEFAULT]
user=ocid1.user.oc1..여기에_복사한_user_OCID
fingerprint=여기에_복사한_fingerprint
tenancy=ocid1.tenancy.oc1..여기에_복사한_tenancy_OCID
region=ap-seoul-1
key_file=~/.oci/oci_api_key.pem
EOF

chmod 600 ~/.oci/config

```

### 5.5 연결 테스트

```bash
oci iam region list --output table

```

성공 시 리전 목록이 출력됩니다.

## 6. VPC, Subnet 생성

ARM 인스턴스를 생성하려면 네트워크 인프라가 필요합니다. VCN(Virtual Cloud Network)과 Subnet을 생성합니다.

### 6.1 VCN 마법사로 생성 (권장)

가장 간단한 방법은 OCI 콘솔의 VCN 마법사를 사용하는 것입니다.

```plain text
☰ (햄버거 메뉴) → Networking → Virtual Cloud Networks → Start VCN Wizard
```

#### 마법사 옵션 선택

"Create VCN with Internet Connectivity" 선택 → Start VCN Wizard

#### 설정값

| 항목 | 값 |
| --- | --- |
| VCN Name | `vcn-arm-free` |  (원하는 이름) |
| Compartment | root compartment 선택 |
| VCN CIDR Block | `10.0.0.0/16` |  (기본값 사용) |
| Public Subnet CIDR | `10.0.0.0/24` |  (기본값 사용) |
| Private Subnet CIDR | `10.0.1.0/24` |  (기본값 사용) |
| →  | **Next** |  →  | **Create** |  클릭 |  |

### 6.2 생성 확인

마법사가 자동으로 생성하는 리소스

- ✅ VCN

- ✅ Public Subnet (인터넷 접속용)

- ✅ Private Subnet

- ✅ Internet Gateway

- ✅ NAT Gateway

- ✅ Service Gateway

- ✅ Route Tables

- ✅ Security Lists

### 6.3 Security List 설정 (SSH 허용)

기본적으로 SSH(22번 포트)는 허용되어 있습니다. 확인 방법:

```plain text
☰ → Networking → Virtual Cloud Networks → [생성한 VCN] 클릭
→ Security Lists → Default Security List 클릭

```

#### Ingress Rules에 다음이 있는지 확인:

| Source | Protocol | Destination Port |
| --- | --- | --- |
| 0.0.0.0/0 | TCP | 22 |

[object Promise]### 6.4 CLI로 생성하는 방법 (선택)

마법사 대신 CLI로 직접 생성할 수도 있습니다:

```bash
# Tenancy ID 확인
TENANCY_ID=$(grep tenancy ~/.oci/config | head -1 | cut -d'=' -f2)
# VCN 생성
VCN_ID=$(oci network vcn create \\
    --compartment-id "$TENANCY_ID" \\
    --display-name "vcn-arm-free" \\
    --cidr-blocks '["10.0.0.0/16"]' \\
    --query "data.id" --raw-output)
echo "VCN ID: $VCN_ID"
# Internet Gateway 생성
IGW_ID=$(oci network internet-gateway create \\
    --compartment-id "$TENANCY_ID" \\
    --vcn-id "$VCN_ID" \\
    --display-name "igw-arm-free" \\
    --is-enabled true \\
    --query "data.id" --raw-output)
echo "Internet Gateway ID: $IGW_ID"
# Route Table 업데이트 (인터넷 라우팅)
RT_ID=$(oci network route-table list \\
    --compartment-id "$TENANCY_ID" \\
    --vcn-id "$VCN_ID" \\
    --query "data[0].id" --raw-output)
oci network route-table update \\
    --rt-id "$RT_ID" \\
    --route-rules "[{\\"destination\\": \\"0.0.0.0/0\\", \\"networkEntityId\\": \\"$IGW_ID\\"}]" \\
    --force
# Subnet 생성
SUBNET_ID=$(oci network subnet create \\
    --compartment-id "$TENANCY_ID" \\
    --vcn-id "$VCN_ID" \\
    --display-name "subnet-arm-free" \\
    --cidr-block "10.0.0.0/24" \\
    --query "data.id" --raw-output)
echo "Subnet ID: $SUBNET_ID"
```

### 6.5 생성 완료 확인

```bash
# Subnet 목록 확인
oci network subnet list \\
    --compartment-id $(grep tenancy ~/.oci/config | head -1 | cut -d'=' -f2) \\
    --query "data[].{name:\\"display-name\\", id:id}" \\
    --output table

```

성공 시 생성한 Subnet이 목록에 표시됩니다.

## 7. 자동 재시도 스크립트

유료 계정이어도 "Out of Capacity" 에러가 발생할 수 있습니다. 이를 위해 5분마다 생성을 재시도하는 스크립트입니다.

[object Promise]### 7.1 자동화 스크립트

```bash
cat > ~/oci-a1-retry.sh << 'SCRIPT'
#!/bin/bash

set -euo pipefail

OCI_CONFIG="${OCI_CLI_CONFIG_FILE:-$HOME/.oci/config}"
OCI_PROFILE="${OCI_CLI_PROFILE:-DEFAULT}"
OCI_DIR="$HOME/.oci"
LOG_FILE="$OCI_DIR/a1-retry.log"
SSH_KEY_PRIVATE="$OCI_DIR/oci-a1-key"
SSH_KEY_PUBLIC="$OCI_DIR/oci-a1-key.pub"

INSTANCE_NAME="${INSTANCE_NAME:-arm-ubuntu-24}"
OCPUS="${OCPUS:-4}"
MEMORY_GB="${MEMORY_GB:-24}"
BOOT_VOLUME_GB="${BOOT_VOLUME_GB:-50}"
RETRY_INTERVAL="${RETRY_INTERVAL:-300}"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

notify_success() {
    if command -v osascript &> /dev/null; then
        osascript -e 'display notification "OCI A1 인스턴스 생성 성공! 🎉" with title "OCI 알림" sound name "Glass"'
        for _ in {1..3}; do
            afplay /System/Library/Sounds/Glass.aiff 2>/dev/null || true
            sleep 1
        done
    fi
}

notify_error() {
    if command -v osascript &> /dev/null; then
        osascript -e 'display notification "OCI A1 스크립트 에러 발생!" with title "OCI 에러" sound name "Basso"'
    fi
}

parse_oci_config() {
    local key="$1"
    awk -v profile="[$OCI_PROFILE]" -v key="$key" '
        $0 == profile { found=1; next }
        /^\\[/ { found=0 }
        found && $0 ~ "^"key"=" { gsub(/.*=[ ]*/, ""); print; exit }
    ' "$OCI_CONFIG"
}

generate_ssh_key() {
    if [[ -f "$SSH_KEY_PRIVATE" ]]; then
        log "기존 SSH 키 사용: $SSH_KEY_PRIVATE"
    else
        log "새 SSH 키 생성 중..."
        ssh-keygen -t ed25519 -f "$SSH_KEY_PRIVATE" -N "" -C "oci-a1-instance"
        chmod 600 "$SSH_KEY_PRIVATE"
        chmod 644 "$SSH_KEY_PUBLIC"
        log "SSH 키 생성 완료"
    fi
}

if [[ ! -f "$OCI_CONFIG" ]]; then
    echo "❌ OCI 설정 파일이 없습니다: $OCI_CONFIG"
    echo "먼저 'oci setup config' 를 실행하세요."
    exit 1
fi

log "=== OCI 설정 로드 중 ==="

TENANCY_ID=$(parse_oci_config "tenancy")
REGION=$(parse_oci_config "region")

if [[ -z "$TENANCY_ID" || -z "$REGION" ]]; then
    log "❌ ~/.oci/config 에서 tenancy/region을 찾을 수 없습니다."
    exit 1
fi

log "Tenancy: ${TENANCY_ID:0:30}..."
log "Region: $REGION"

log "Availability Domain 조회 중..."
AD=$(oci iam availability-domain list --query "data[0].name" --raw-output 2>/dev/null)
log "AD: $AD"

log "Subnet 조회 중..."
SUBNET_ID=$(oci network subnet list --compartment-id "$TENANCY_ID" --query "data[0].id" --raw-output 2>/dev/null)
if [[ -z "$SUBNET_ID" || "$SUBNET_ID" == "null" ]]; then
    log "❌ Subnet을 찾을 수 없습니다. VCN/Subnet을 먼저 생성하세요."
    exit 1
fi
log "Subnet: ${SUBNET_ID:0:50}..."

log "Ubuntu ARM 이미지 조회 중..."
IMAGE_ID=$(oci compute image list \\
    --compartment-id "$TENANCY_ID" \\
    --shape "VM.Standard.A1.Flex" \\
    --query "data[?contains(\\"display-name\\", 'Canonical-Ubuntu-24.04-aarch64')] | [0].id" \\
    --raw-output 2>/dev/null)
if [[ -z "$IMAGE_ID" || "$IMAGE_ID" == "null" ]]; then
    IMAGE_ID=$(oci compute image list \\
        --compartment-id "$TENANCY_ID" \\
        --shape "VM.Standard.A1.Flex" \\
        --query "data[?contains(\\"display-name\\", 'Ubuntu')] | [0].id" \\
        --raw-output 2>/dev/null)
fi
log "Image: ${IMAGE_ID:0:50}..."

generate_ssh_key

log "=== 자동 재시도 시작 ==="
log "간격: ${RETRY_INTERVAL}초 | 스펙: ${OCPUS} OCPU / ${MEMORY_GB}GB RAM"

attempt=0
while true; do
    ((attempt++))
    log "시도 #$attempt..."

    result=$(oci compute instance launch \\
        --compartment-id "$TENANCY_ID" \\
        --availability-domain "$AD" \\
        --shape "VM.Standard.A1.Flex" \\
        --shape-config "{\\"ocpus\\": $OCPUS, \\"memoryInGBs\\": $MEMORY_GB}" \\
        --display-name "$INSTANCE_NAME" \\
        --image-id "$IMAGE_ID" \\
        --subnet-id "$SUBNET_ID" \\
        --boot-volume-size-in-gbs "$BOOT_VOLUME_GB" \\
        --assign-public-ip true \\
        --ssh-authorized-keys-file "$SSH_KEY_PUBLIC" \\
        2>&1) || true

    if echo "$result" | grep -q '"lifecycle-state"'; then
        instance_id=$(echo "$result" | grep '"id"' | head -1 | cut -d'"' -f4)
        log "✅ 성공! Instance ID: $instance_id"
        log "시도 횟수: $attempt"
        log ""
        log "============================================"
        log "SSH 접속: ssh -i $SSH_KEY_PRIVATE ubuntu@<PUBLIC_IP>"
        log "IP 확인: oci compute instance list-vnics --instance-id $instance_id --query \\"data[0].\\\\\\"public-ip\\\\\\"\\" --raw-output"
        log "============================================"
        notify_success
        exit 0

    elif echo "$result" | grep -q "Out of host capacity"; then
        log "⏳ 용량 부족. ${RETRY_INTERVAL}초 후 재시도..."

    elif echo "$result" | grep -q "TooManyRequests"; then
        log "⚠️ Rate limit. 10분 대기..."
        sleep 600
        continue

    else
        log "❌ 에러: $result"
        notify_error
        exit 1
    fi

    sleep "$RETRY_INTERVAL"
done
SCRIPT

chmod +x ~/oci-a1-retry.sh

```

### 7.2 실행 스크립트 생성 (macOS 슬립 방지)

```bash
cat > ~/oci-a1-start.sh << 'SCRIPT'
#!/bin/bash

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"

if command -v caffeinate &> /dev/null; then
    caffeinate -s "$SCRIPT_DIR/oci-a1-retry.sh"
else
    "$SCRIPT_DIR/oci-a1-retry.sh"
fi
SCRIPT

chmod +x ~/oci-a1-start.sh

```

### 7.3 스크립트 실행

```bash
# Foreground 실행 (터미널에서 직접 확인)
~/oci-a1-start.sh

# Background 실행 (터미널 닫아도 계속, macOS 슬립 방지)
nohup ~/oci-a1-start.sh &

# 로그 확인
tail -f ~/.oci/a1-retry.log

```

### 7.4 스펙 변경 (선택)

환경 변수로 스펙을 조정할 수 있습니다.

오라클 공식 문서에서는 여러 인스턴스를 합해서 200gb까지는 부트 볼륨 크기가 무료인데, 가용성에 따라 기습적으로 과금이 일어나는 일이 많은 것 같습니다. 50gb가 안전합니다. 

