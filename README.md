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
## 🛠️ 환경 설정 및 버전

 Control-plane(터틀봇 원격 PC) 의 주요 소프트웨어 및 버전 정보

| 항목                  | 버전 / 세부 정보                         |
| --------------------- | --------------------------------------- |
| **ROS 2**             | ros2-jazzy                              |
| **Kernel**            | Linux 6.11.0-26-generic                |
| **Architecture**      | x86-64                                  |
| **Operating System**  | Ubuntu 24.04.2 LTS                      |
| **k3s**               | v1.32.5+k3s1                            |
| **Container Runtime** | containerd://2.0.5-k3s1.32              |

## 컴포넌트 역할

| 컴포넌트                 | 설명                                                      |
| :--------------------- | :-------------------------------------------------------- |
| 🗓️ **sdi-scheduler**     | turtlebot 배터리(wh) 및 위지정보 기반 커스텀 스케줄러           |
| 🔄 **metric-collector**  | TurtleBot 및 시스템 메트릭을 Metric-Collector 큐에 적재하는 모듈      |
| 🗄️ **influxdb**           | 수집된 시계열 메트릭을 영구 저장하는 InfluxDB 데이터베이스     |
| 📥 **metrics-ingester**   | Metric-Collector 큐에서 메트릭 메시지를 소비해 InfluxDB에 기록         |
| 🔎 **analysis-engine**    | 다양한 로그 및 메트릭 분석 엔진   |
| ⚖️ **policy-engine**      | 분석 결과 기반으로 MALE 정책을 적용 시스템 동작을 제어하는 엔진 |


## 오케스트레이션 구성

### K3s 설치(필수)&#x20;

```bash
# Control‑Plane에서 실행
curl -sfL https://get.k3s.io | sh -
sudo cat /var/lib/rancher/k3s/server/node-token  # 토큰 복사

# Worker(Node/TurtleBot)에서 실행, CONTROL-PLANE_IP, CONTROL-PLANE_TOKEN 값 명령어에 넣기
curl -sfL https://get.k3s.io | K3S_URL=https://<CONTROL‑PLANE_IP>:6443 K3S_TOKEN=<CONTROL‑PLANE_TOKEN> sh -
```

### K9s 설치(선택)&#x20;

파드들의 정보를 손쉽게 확인하기 위해 KETI에서는 설치했습니다. 필수가 아닌 선택입니다.

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

# 주석 “직접 설정” 적힌 부분(12·13·21·22행) id,pw 설정
vi tbot-monitoring.yaml
```

#### 배포

```bash
kubectl apply -f tbot-monitoring.yaml
```

컴포넌트 상태는 다음과 같이 확인할 수 있습니다.

```bash
kubectl get pods -n tbot-monitoring # 또는 k9s
```

<img src="https://github.com/user-attachments/assets/aadcf3e1-9559-4a5f-856a-77d2cc46bf3e" width="600" height="105"/>
---

### 스케줄러 컴포넌트&#x20;

#### InfluxDB 토큰 확인 절차

 브라우저에서 `http://<CONTROL‑PLANE_IP>:32086` 접속 이후 아래 이미지 대로 따라가시면 됩니다.

 <img src="https://github.com/user-attachments/assets/f8725d74-a945-40b3-8b5d-c92d6cfbe68a" width="600" height="993"/>



#### 배포 파일 수정

```bash
cd ../../scheduler
vi sdi-scheduler-deploy.yaml   # 43행 주석에“직접 설정” 적혀있는 부분에 복사한 토큰 값 넣기
```

#### 배포

```bash
kubectl apply -f sdi-scheduler-deploy.yaml
```

상태 확인:

```bash
kubectl get pod -n kube-system # 또는 k9s
```
<img src="https://github.com/user-attachments/assets/ff5c0f17-8394-4487-932e-1e90a319f122" width="600" height="197"/>



스케줄링시 로그(로그 확인 방법 하단 기술)
<img src="https://github.com/user-attachments/assets/9ef7b440-9236-412a-9f19-db619aaa80d8" width="867" height="55"/>


#### 스케줄러 사용법
sdi-scheduler-test.yaml 파일 6번째줄 처럼 schedulerName: `schedulerName: sdi-scheduler`를 적고 사용하면됩니다.(주석 확인)


### 오케스트레이션 엔진&#x20;

#### 배포
```bash
cd ../engine
kubectl apply -f orchestration-engines-deploy.yaml

```

상태 확인:
```bash
kubectl get pod -n orchestration-engines   # 또는 k9s
```
<img src="https://github.com/user-attachments/assets/06315652-55db-4e3f-abb3-a73b3b4c914b" width="600" height="70"/>

---

## 로그 확인&#x20;
> **Tip** 모든 컴포넌트가 Deployment 형태로 실행되므로 자동 생성된 파드 이름 추정할 수 없어 로그 확인 방법을 추가로 작성합니다.

| 방법      | 명령                                   |
| ------- | ------------------------------------ |
| kubectl | `kubectl logs -f <파드이름> -n <네임스페이스>` |
| k9s     | 파드 선택 후 **Space** → **L**            |

---



