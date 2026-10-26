# 자폭드론 
ASTRO (정연수(202255671) , 김현태(202055529))

# 재난자 구조 및 자폭 드론 프로젝트

## 1. 프로젝트 소개
- **프로젝트명:** 재난자 구조 및 자폭 드론 (Rescue & Self-Destruct Drone)  
- **목적:**  
  재난 현장에서 신속히 생존자를 탐지하고 구조 상황을 지원하며, 위험한 상황에서는 자폭 기능을 통해 2차 피해를 방지하는 것을 목표로 한다.  
- **개요:**  
  본 프로젝트는 **Intel RealSense 카메라 + Jetson + Pixhawk** 기반의 팀 프로젝트다.  
  **Nano와 Xavier 환경을 사용했으며, 추론 최적화는 Jetson Xavier·TensorRT FP16 기준이다.**  
  - **YOLOv5 모델**을 통한 실시간 객체 인식  
  - **Pixhawk**를 통한 드론 모터 제어  
  - **Mission Planner**를 통한 자동 비행 및 Guided 모드 전환  
  - **UDP 스트리밍**을 통한 실시간 영상 전송 및 원격 제어 지원  

---

## 2. 팀 소개
| 이름       | 이메일                  | 역할                          |
|------------|-------------------------|-------------------------------|
| 정연수     | aden1213@pusan.ac.kr   | 프로젝트 총괄, 드론 하드웨어 구성 , VPN 네트워크 구성|
| 김현태     | tee9665@pusan.ac.kr   | Jetson Nano·Xavier 환경 설정, YOLOv5n 구성·TensorRT FP16 적용, VPN 네트워크 구성|

1. **Camera (Intel RealSense D435i)** → 타겟 인식  
2. **Jetson (Nano·Xavier / 최적화: Xavier·TensorRT FP16)** → 객체 탐지 및 판단  
3. **Pixhawk (Flight Controller)** → 모터 제어, 비행 모드 전환  
4. **Drone (프레임 + 모터 + 배터리)** → 실제 이동 및 제어  
5. **Ground Station (Mission Planner, ffplay)** → 실시간 모니터링 및 제어  


---

## 3. 구성도
### 동작 흐름도
<img width="1024" height="498" alt="image" src="https://github.com/user-attachments/assets/61c8828c-280b-4e1a-9ade-bca0df6b73e4" />


## 4. 네트워크 
본 프로젝트는 **재난 현장 원격 제어 및 모니터링**을 위해 네트워크 기반 구조를 갖춘다.  
- **UDP 스트리밍**: 지연(latency)을 최소화하기 위해 TCP 대신 UDP 프로토콜을 사용한다.  
- **Ground Station ↔ Drone**: 상호 간의 명령 및 영상 데이터 교환은 **양방향 통신**을 기반으로 한다.  
- **실시간 영상 전송**: Jetson에서 YOLO 객체 인식 결과를 인코딩(H.264)하여 Ground Station으로 송출한다.  
- **원격 제어**: Ground Station에서 Mission Planner를 통해 Pixhawk로 명령 전송.  

<img width="819" height="546" alt="image" src="https://github.com/user-attachments/assets/0788692c-4c42-4731-892f-8cc91da23c00" />




## 5. YOLOv5n·TensorRT FP16 적용

- **YOLOv5n/TensorRT 엔진**을 활용하여 실시간으로 재난 현장의 타겟(사람, 물체 등)을 탐지한다.  
- **TensorRT 적용 환경**: Jetson Xavier에서 YOLOv5n에 TensorRT FP16을 적용했다.  
- **탐지 결과 활용**:  
  - 탐지된 객체의 **Bounding Box 중심 좌표**를 계산  
  - 해당 좌표값을 Pixhawk로 전달하여 드론의 위치 제어에 반영  
- **추가 기능**:  
  - YOLO Confidence 값 기반으로 **임계값(Threshold 70%)**을 설정  
  - 일정 시간(예: 2초 이상) 조건 충족 시 **Guided 모드 전환** 및 **모터 제어** 수행  

## 사용 장비·정밀도와 검증 범위

| 항목 | 실제 사용 범위 |
|---|---|
| 장비 환경 설정 | Jetson Nano·Xavier |
| 추론 최적화 장비 | NVIDIA Jetson Xavier |
| 인식 모델 | YOLOv5n |
| 추론 실행 방식 | TensorRT FP16 |
| 본인 담당 | Jetson 환경 설정, YOLOv5n 구성·TensorRT FP16 적용 |
| 팀 시스템 | 카메라 입력·전처리·추론·박스 표시·영상 전송과 기체 제어 연결 |

장비 환경 설정 경험과 **Xavier에서 FP16을 적용한 경험**을 구분한다. 카메라·통신·제어를 포함한 전체 시스템은 팀 결과물이다.

현재 공개 자료만으로는 원시 측정 로그와 반복 시험 조건을 확인할 수 없어, **FPS·GPU/메모리 사용량의 정량적 개선은 주장하지 않는다.** 기존 성능 비교 수치와 해당 비교 그래프는 제외했다. 아래 실행 화면과 영상은 동작 사례이며, 특정 장비·정밀도의 성능 측정 결과를 뜻하지 않는다.

- **YOLO 돌린 화면**:
<img width="881" height="468" alt="image" src="https://github.com/user-attachments/assets/232d2e87-bb40-4a05-b614-5e6ba4c5e8f8" />
- **드론에서 YOLO 돌리고 Target 발견뒤 공중 hovering(제자리 멈춤) 스크린샷**:
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/3977eaf9-5142-43e4-86c6-29f879d5694b" />

# 실행방법 · 기존 Nano 환경 예시 
1. window 컴퓨터에서 window_computer/color.sdp 파일을 cmd를 키고 실행한다 , 그리고 jetson nano 로 원격접속후 , code/perfect_hold_display.py 파일을 python python perfect_hold_display.py 로 실행하면 udp streaming 화면이 나온다


# 유튜브 링크 : Nano 환경 드론 카메라 시점 
  https://www.youtube.com/watch?v=Vai7mIXLj4c

# 유튜브 링크 : target 인식 후 자동 하강 , 옆에서 본 드론 움직임 
 https://youtu.be/RfEyx5kR5wU

# 유튜브 링크 : Gazebo 를 이용한 시뮬레이션 + mission planner 도 함께 
https://www.youtube.com/watch?v=XjWCKADeDpY


