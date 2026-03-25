# OpenRatSLAM2 — ROS 2 Humble (Ubuntu 22.04)

ROS 2 Humble 환경에 맞춰 포팅한 OpenRatSLAM2 패키지입니다.
원본 저장소: [OpenRatSLAM2/ratslam](https://github.com/OpenRatSLAM2/ratslam)

---

## 개발 환경

| 항목 | 버전 |
|------|------|
| OS | Ubuntu 22.04 LTS |
| ROS | ROS 2 Humble Hawksbill |
| CMake | 3.8+ |
| C++ | C++14 |
| OpenCV | 4.x |
| Boost | 1.74+ |
| Irrlicht | 1.8+ |

---

## 의존성 설치

```bash
sudo apt update
sudo apt install -y \
  ros-humble-cv-bridge \
  ros-humble-sensor-msgs \
  ros-humble-geometry-msgs \
  ros-humble-nav-msgs \
  ros-humble-visualization-msgs \
  libboost-serialization-dev \
  libopencv-dev \
  libirrlicht-dev \
  libopengl-dev
```

---

## 빌드 방법

```bash
# 1. 워크스페이스 생성
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

# 2. 저장소 클론 (ratslam + topological_msgs 포함)
git clone https://github.com/Dogeun99/OpenRatSLAM2_final.git .

# 3. ROS 2 Humble 환경 설정
source /opt/ros/humble/setup.bash

# 4. 빌드
cd ~/ros2_ws
colcon build --symlink-install

# 5. 워크스페이스 소싱
source ~/ros2_ws/install/setup.bash
```

---

## 데이터셋

To use this package with your datasets, first convert the dataset to a ROS 2 bag format, if necessary. Access the datasets google drive link https://drive.google.com/drive/folders/1ggAxMzIyadmenUPoAon2rL8gvJvuv75K?usp=drive_link for examples.

| 데이터셋 | 설명 | topic_root |
|----------|------|------------|
| St Lucia | 야외 도로 주행 환경 | `stlucia` |
| iRat Australia | 소형 로봇 실내 환경 | `irat_red` |
| Oxford New College | 캠퍼스 야외 환경 (파노라마) | `/newcollege` |

---

## 실행 방법

각 데이터셋에 대해 **터미널 2개**를 사용합니다.

### 1. St Lucia

**터미널 1 — RatSLAM 노드 실행:**
```bash
source ~/ros2_ws/install/setup.bash
ros2 launch ratslam stlucia.launch
```

**터미널 2 — bag 재생:**
```bash
ros2 bag play <stlucia_dataset_path>.db3 \
  --rate 1.0 \
  --clock \
  --start-paused \
  --topics /stlucia/image/compressed
```

---

### 2. iRat Australia

**터미널 1 — RatSLAM 노드 실행:**
```bash
source ~/ros2_ws/install/setup.bash
ros2 launch ratslam irataus.launch
```

**터미널 2 — bag 재생:**
```bash
ros2 bag play <irat_aus_dataset_path>.db3 \
  --rate 1.0 \
  --clock \
  --start-paused \
  --topics /irat_red/odom /irat_red/camera/image/compressed
```

---

### 3. Oxford New College

**터미널 1 — RatSLAM 노드 실행:**
```bash
source ~/ros2_ws/install/setup.bash
ros2 launch ratslam oxford_newcollege.launch
```

**터미널 2 — bag 재생:**
```bash
ros2 bag play <oxford_newcollege_dataset_path>.db3 \
  --rate 1.0 \
  --clock \
  --start-paused \
  --topics /newcollege/image/compressed
```

---

## 패키지 구조

```
src/
├── ratslam/               # 메인 RatSLAM 패키지
│   ├── config/            # 데이터셋별 파라미터 설정
│   ├── launch/            # 데이터셋별 launch 파일
│   ├── media/             # 시각화용 텍스처
│   ├── src/
│   │   ├── ratslam/       # 핵심 알고리즘 (LV, PC, EM, VO)
│   │   ├── graphics/      # Irrlicht 기반 시각화
│   │   └── main_*.cpp     # 각 노드 진입점
│   └── CMakeLists.txt
└── topological_msgs/      # 커스텀 메시지 패키지
    └── msg/               # TopologicalMap, Edge, Node 등
```

---

## 라이선스

GPL-3.0 — 원본 저작자: João Victor Torres Borges
