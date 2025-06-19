# ☁️ KETI Orchestration 환경 설정 가이드


## 📋 목차

1. [오케스트레이션 구성](#오케스트레이션-구성)
   1. [K3s 설치(필수)](#k3s-설치필수-)
   2. [K9s 설치(선택)](#k9s-설치선택-)
2. [컴포넌트 설치](#컴포넌트-설치)
   1. [프로파일링 컴포넌트](#프로파일링-컴포넌트-)
   2. [스케줄러 컴포넌트](#스케줄러-컴포넌트-)
   3. [오케스트레이션 엔진](#오케스트레이션-엔진-)
3. [로그 확인](#로그-확인)

---

## 오케스트레이션 구성

### K3s 설치(필수)&#x20;

```bash
# Control‑Plane에서 실행
autoChristopher
curl -sfL https://get.k3s.io | sh -
sudo cat /var/lib/rancher/k3s/server/node-token  # 토큰 복사

# Worker(Node/TurtleBot)에서 실행
curl -sfL https://get.k3s.io | K3S_URL=https://<CONTROL‑PLANE_IP>:6443 K3S_TOKEN=<CONTROL‑PLANE_TOKEN> sh -
```

### K9s 설치(선택)&#x20;

```bash
mkdir k9s && cd k9s
wget https://github.com/derailed/k9s/releases/download/v0.26.7/k9s_Linux_x86_64.tar.gz
tar zxvf k9s_Linux_x86_64.tar.gz
sudo mv k9s /usr/local/bin/
cp /etc/rancher/k3s/k3s.yaml ~/.kube/config  # k9s에서 k3s 클러스터 조회 가능
```

`k9s` 명령어를 입력해 UI가 실행되면 정상 설치된 것입니다.

---

## 컴포넌트 설치

### 프로파일링 컴포넌트&#x20;

```bash
git clone https://github.com/sungmin306/SDI-Orchestration.git
cd SDI-Orchestration/profiling/manifest/

# 주석 “직접 설정” 적힌 부분(12·13·21·22행) 편집
vi tbot-monitoring.yaml
```

#### 배포

```bash
kubectl apply -f tbot-monitoring.yaml
```

컴포넌트 상태는 다음과 같이 확인할 수 있습니다.

```bash
kubectl get pods -n tbot-monitoring
```

<img src="https://github.com/user-attachments/assets/aadcf3e1-9559-4a5f-856a-77d2cc46bf3e" width="748" height="131"/>
---

### 스케줄러 컴포넌트&#x20;

#### InfluxDB 토큰 확인 절차

 브라우저에서 `http://<CONTROL‑PLANE_IP>:32086` 접속 이후 아래 이미지 대로 따라가시면 됩니다.
<img src="https://github.com/user-attachments/assets/bd9fa5b8-2ce1-4704-a9cd-590c889259ed" width="504" height="276"/>
<img src="https://github.com/user-attachments/assets/d9e2fdf1-e38a-4c3a-a3d1-0e9dea6f1261" width="540" height="360"/>
<img src="https://github.com/user-attachments/assets/df99a4c5-32d4-400e-bd7d-6a42f018b341" width="574" height="197"/>

#### 배포 파일 수정

```bash
cd ../../scheduler
vi sdi-scheduler-deploy.yaml   # 43행 주석에“직접 설정” 적혀있음
```

#### 배포

```bash
kubectl apply -f sdi-scheduler-deploy.yaml
```

상태 확인:

```bash
kubectl get pod -n kube-system
```
<img src="https://github.com/user-attachments/assets/ff5c0f17-8394-4487-932e-1e90a319f122" width="567" height="186"/>
---

### 오케스트레이션 엔진&#x20;

```bash
cd ../engine
kubectl apply -f orchestration-engines-deploy.yaml

```

상태 확인:
```bash
kubectl get pod -n orchestration-engines   # 또는 k9s
```
<img src="https://github.com/user-attachments/assets/06315652-55db-4e3f-abb3-a73b3b4c914b" width="512" height="60"/>

---

## 로그 확인&#x20;
> **Tip** 모든 컴포넌트가 Deployment 형태로 실행되므로 자동 생성된 파드 이름 추정할 수 없어 로그 확인 방법을 추가로 작성합니다.

| 방법      | 명령                                   |
| ------- | ------------------------------------ |
| kubectl | `kubectl logs -f <파드이름> -n <네임스페이스>` |
| k9s     | 파드 선택 후 **Space** → **L**            |

---



