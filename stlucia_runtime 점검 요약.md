# St Lucia 2007 (ROS 2 Humble 포팅본) 재현성 점검 요약

## 1. 핵심 결론

현재 **St Lucia 실행 경로, 토픽 연결, 핵심 파라미터 적용은 논문과 대체로 일치**한다.  
또한 현재 보인 **`Templates ≈ Iteration` 패턴만으로는 Local View 재사용 실패라고 볼 수 없다.**

가장 중요한 해석은 다음 두 가지다.

1. **`Iteration` 로그는 프레임 번호가 아니라, 현재 프레임에서 비교한 template 수**다.  
   따라서 `Templates: 5446, Iteration: 5446` 같은 패턴은 구조적으로 자연스럽다.
2. 실제 로그 샘플에서는 **`VTM[...]`(기존 template 재사용) 이벤트가 `VTN[...]`(새 template 생성)보다 더 많이 나타났다.**  
   따라서 샘플 구간 기준으로는 **Local View 재사용이 실제로 일어나고 있다**고 보는 쪽이 맞다.

---

## 2. 최종 판단 한 줄 요약

- **정상 여부:** 현재 확보한 근거만으로는 **비정상이라고 단정하기 어렵다**.
- **논문 정합성:** 현재 확인된 범위에서는 **논문과 대체로 정합적**이다.
- **가장 가능성 높은 오해 지점:** **`Templates / Iteration` 로그 의미를 잘못 읽은 것**.

---

## 3. 논문 대비 파라미터 점검 결과

논문 기준은 다음을 참조했다.
- **OpenRatSLAM: an open source brain-based SLAM system**
- **Table 4 / Section 4.2 / Section 6.1**

### 3-1. Local View

| 파라미터 | 논문 | config | runtime | 판정 |
|---|---:|---:|---:|---|
| `vt_match_threshold` | 0.085 | 0.085 | 0.085 | 일치 |
| `template_x_size` | 60 | 60 | 60 | 일치 |
| `template_y_size` | 10 | 10 | 10 | 일치 |
| `image_crop_x_min` | 40 | 40 | 40 | 일치 |
| `image_crop_x_max` | 600 | 600 | 600 | 일치 |
| `image_crop_y_min` | 150 | 150 | 150 | 일치 |
| `image_crop_y_max` | 300 | 300 | 300 | 일치 |
| `vt_shift_match` | 5 | 5 | 5 | 일치 |
| `vt_step_match` | 1 | 1 | 1 | 일치 |
| `vt_panoramic` | 0 | 없음 | 0 | 기본값 fallback, 논문과 정합 |
| `vt_normalisation` | 0.4 | 0.4 | 0.4 | 일치 |
| `vt_patch_normalisation` | 0 | 없음 | 0 | 기본값 fallback, 논문과 정합 |

### 3-2. Visual Odometry

| 파라미터 | 논문 | config | runtime | 판정 |
|---|---:|---:|---:|---|
| `camera_hz` | 10 | 10.0 | 10.0 | 일치 |
| `camera_fov_deg` | 53 | 53.0 | 53.0 | 일치 |
| `vtrans_scaling` | 1000 | 1000.0 | 1000.0 | 일치 |
| `vtrans_max` | 20 | 20.0 | 20.0 | 일치 |

### 3-3. Pose Cells / Experience Map

| 파라미터 | 논문 | config | runtime | 판정 |
|---|---:|---:|---:|---|
| `pc_vt_inject_energy` | 0.15 | 0.15 | 0.15 | 일치 |
| `pc_cell_x_size` | 2.0 | 2.0 | 2.0 | 일치 |
| `pc_dim_xy` | 30 | 30 | 30 | 일치 |
| `vt_active_decay` | 1.0 | 1.0 | 1.0 | 일치 |
| `pc_vt_restore` | 0.04 | 0.04 | 0.04 | 일치 |
| `exp_delta_pc_threshold` | 1.0 | 1.0 | `/ratslam_pose_cells`에서 1.0 | 일치 |
| `exp_loops` | 20 | 20 | 20 | 일치 |
| `exp_initial_em_deg` | 90 | 없음 | 90.0 | 기본값 fallback, 논문과 정합 |
| `exp_correction` | 0.5 | 없음 | 0.5 | 기본값 fallback, 논문과 정합 |

### 3-4. 시간 동기화

| 항목 | 확인 결과 | 판정 |
|---|---|---|
| `use_sim_time` | 4개 노드 모두 `True` | 정상 |
| `/clock` | bag play 중 publisher 1개, 약 20 Hz | 정상 |

### 3-5. 파라미터 관련 핵심 결론

- **YAML namespace mismatch 없음**
- **node name mismatch 없음**
- **launch wiring 문제 없음**
- **핵심 St Lucia 파라미터 누락 없음**
- 기본값 fallback 항목은 존재하지만, **논문값과 충돌하는 항목은 확인되지 않음**

---

## 4. launch → yaml → node 흐름

실행 엔트리포인트는 `stlucia.launch` 이다.

| 노드 | 실행명 | YAML section | launch 직접 주입 | 비고 |
|---|---|---|---|---|
| `/ratslam_view_template` | `ratslam_lv` | `ratslam_view_template` | `use_sim_time=true`, `topic_root=stlucia` | `vt_panoramic`, `vt_patch_normalisation` 등은 코드 기본값 fallback |
| `/ratslam_visual_odometry` | `ratslam_vo` | `ratslam_visual_odometry` | `use_sim_time=true`, `topic_root=stlucia` | 핵심 VO 값은 대부분 YAML에 존재 |
| `/ratslam_pose_cells` | `ratslam_pc` | `ratslam_pose_cells` | `use_sim_time=true`, `topic_root=stlucia`, `media_path`, `image_file` | 일부 PC 내부 상수는 코드 기본값 |
| `/ratslam_experience_map` | `ratslam_em` | `ratslam_experience_map` | `use_sim_time=true`, `topic_root=stlucia`, `media_path`, `image_file` | `exp_correction`, `exp_initial_em_deg`는 코드 기본값 fallback |

### 요약

실제 runtime 확인 결과,

- **launch → yaml → node parameter 주입 경로는 정상**
- **핵심 파라미터가 launch 단계에서 빠져 잘못된 기본값으로 떨어지는 현상은 없음**

---

## 5. bag / node / topic 연결 점검

### 5-1. 핵심 토픽 매칭

| 토픽 | 타입 | 누가 publish | 누가 subscribe | 판정 |
|---|---|---|---|---|
| `/stlucia/camera/image/compressed` | `sensor_msgs/msg/CompressedImage` | bag player | LV, VO | 정상 |
| `/stlucia/odom` | `nav_msgs/msg/Odometry` | VO | Pose Cells, Experience Map | 정상 |
| `/clock` | `rosgraph_msgs/msg/Clock` | rosbag2_player | 4개 노드 전체 | 정상 |
| `/stlucia/view_template` | `topological_msgs/msg/ViewTemplate` | LV | Pose Cells | 정상 |
| `/stlucia/PoseCell/TopologicalAction` | `topological_msgs/msg/TopologicalAction` | Pose Cells | Experience Map | 정상 |

### 5-2. 핵심 확인 결과

- `ros2 bag info` 기준, bag는 **image-only bag**이며 주요 입력은 `/stlucia/camera/image/compressed`
- `/ratslam_visual_odometry`는 입력 이미지로부터 `/stlucia/odom` 생성
- `/ratslam_pose_cells`는 `/stlucia/odom`, `/stlucia/view_template` 구독
- `/ratslam_experience_map`는 `/stlucia/odom`, `/stlucia/PoseCell/TopologicalAction` 구독

### 결론

- **topic wiring 정상**
- **bag 입력 → VO → Pose Cells / Experience Map 흐름 정상**

---

## 6. /clock / sim time / odom 점검 결과

### 확인 결과

- `/clock`
  - bag play 전: publisher 0, subscriber 4
  - bag play 중: publisher 1, subscriber 4
  - 평균 약 **20 Hz**
- `/stlucia/camera/image/compressed`
  - 평균 약 **10 Hz**
  - `CompressedImage(JPEG)` 실제 수신 확인
- `/stlucia/odom`
  - publisher 1, subscriber 2
  - 평균 약 **10 Hz**
  - `linear.x`, `angular.z`에 실제 값 존재
- 이미지와 odom의 샘플 timestamp가 동일하게 맞는 경우 확인됨

### 결론

- **sim time wiring 정상**
- **/clock 갱신 정상**
- **launch 후 bag play 순서 적절**
- 현재 샘플 기준으로는 **timing 문제가 1차 원인일 가능성이 낮다**
- **VO는 camera rate와 거의 같은 속도로 odom을 잘 생성**하고 있다

---

## 7. Local View 재사용 분석

### 가장 중요한 해석

> `Templates ≈ Iteration` 은 실패의 증거가 아니다.

이유는 다음과 같다.

- `Templates:` 로그는 **현재까지 저장된 template 총 개수**
- `Iteration:` 로그는 **현재 프레임 매칭 시 순회한 template 개수**

즉 저장된 template 수가 많아질수록, 비교 과정에서 순회한 template 수 역시 비슷해지는 패턴은 자연스럽다.

### 실제 로그에서 확인된 점

20초 샘플 `launch.log` 기준:

- `VTN[` 개수: **191**
- `VTM[` 개수: **424**

즉,

- **기존 template 재사용(`VTM`) > 새 template 생성(`VTN`)**
- 따라서 적어도 이 샘플 구간에서는 **Local View 재사용이 존재**한다

### 관찰된 대표 패턴

- 초반: `VTM[0]` 반복 후 `VTN[1]`
- 이후: `VTM[1]`, `VTM[2]` 등의 재사용 구간 반복
- 뒤 구간: `VTM[64]` 반복 후 `VTN[65]`

이 패턴은 오히려 **정상적인 재사용**의 흔적으로 해석하는 편이 맞다.

---

## 8. Experience vs Template 증가 경향

샘플 시점 기준:

- `/stlucia/view_template --once` 에서 `current_id ≈ 712`
- `/stlucia/PoseCell/TopologicalAction --once` 에서 `dest_id ≈ 1132`

대략적인 비율:

- **experience / template ≈ 1132 / 712 ≈ 1.59**

### 해석

논문에서 언급한 것처럼 **experience 수가 visual template 수보다 더 빠르게 증가하는 경향**과 같은 방향이다.

즉 이 관찰은 **논문과 어긋나는 신호라기보다, 오히려 정합적인 신호**에 가깝다.

---

## 9. 논문과 다르게 보일 수 있는 원인 우선순위

### 1위. `Iteration` 로그 해석 오류

가장 가능성이 높다.

- `Iteration`은 frame index가 아님
- template comparison 횟수에 가깝다
- 따라서 `Templates ≈ Iteration`은 place recognition 실패의 직접 증거가 아님

### 2위. 논문 최종 결과와 현재 live 중간 상태를 직접 비교

- 논문 Figure는 **full dataset 종료 후 최종 결과** 성격이 강함
- 현재는 **runtime 일부 구간 샘플** 확인 단계
- 중간 구간만 보고 최종 맵 형상과 바로 비교하면 오해가 생길 수 있음

### 3위. 사용 중인 St Lucia bag variant 차이

- 현재 파일명은 `stlucia_2007-001.db3`
- 논문/기존 공개물은 `stlucia_2007.bag` 계열로 알려진 경우가 많음
- trimming, 재인코딩, 시작/종료 구간 차이가 있으면 최종 결과 차이가 생길 수 있음

### 4위. VO 입력 자체의 미세 차이

- crop, FOV, scaling, rate는 현재 논문과 일치
- odom도 정상 발행 중
- 다만 **full-run 최종 맵 차이가 계속 남는다면** 이후 후보는 VO 쪽 미세 차이일 가능성이 있음

### 5위. 가능성 낮은 항목들

현재 근거상 낮다.

- YAML 미적용
- launch wiring 문제
- topic mismatch
- sim time 문제
- compressed image 처리 문제

---

## 10. 현재 단계의 결론

현재 확보된 런타임 근거를 기준으로 보면:

1. **Humble 포팅본의 St Lucia 실행 경로는 정상에 가깝다.**
2. **핵심 파라미터는 논문과 대체로 일치한다.**
3. **토픽 연결과 시간 동기화도 정상이다.**
4. **VO는 실제로 odom을 안정적으로 생성한다.**
5. **Local View 재사용은 샘플 로그 기준 실제로 존재한다.**
6. 따라서 지금 단계에서는 **“논문과 다르게 동작한다” 또는 “Local View 재사용이 완전히 깨졌다”라고 결론내리기 어렵다.**

---

## 11. 재실행 명령어

### 터미널 1

```bash
source /opt/ros/humble/setup.bash
source /home/asmr/rolling_ws_1/install/setup.bash
ros2 launch ratslam stlucia.launch
```

### 터미널 2

```bash
source /opt/ros/humble/setup.bash
source /home/asmr/rolling_ws_1/install/setup.bash
ros2 bag info /home/asmr/rolling_ws_1/src/ratslam/data/stlucia_2007-001.db3
ros2 bag play /home/asmr/rolling_ws_1/src/ratslam/data/stlucia_2007-001.db3 --clock 20 -r 1.0
```

### 점검 명령어

```bash
source /opt/ros/humble/setup.bash
source /home/asmr/rolling_ws_1/install/setup.bash

ros2 node list
ros2 node info /ratslam_view_template
ros2 node info /ratslam_visual_odometry
ros2 node info /ratslam_pose_cells
ros2 node info /ratslam_experience_map

ros2 param get /ratslam_view_template vt_match_threshold
ros2 param get /ratslam_view_template template_x_size
ros2 param get /ratslam_view_template template_y_size
ros2 param get /ratslam_view_template image_crop_x_min
ros2 param get /ratslam_view_template image_crop_x_max
ros2 param get /ratslam_view_template image_crop_y_min
ros2 param get /ratslam_view_template image_crop_y_max
ros2 param get /ratslam_view_template vt_shift_match
ros2 param get /ratslam_view_template vt_step_match
ros2 param get /ratslam_view_template vt_panoramic

ros2 param get /ratslam_visual_odometry camera_hz
ros2 param get /ratslam_visual_odometry camera_fov_deg
ros2 param get /ratslam_visual_odometry vtrans_scaling
ros2 param get /ratslam_visual_odometry vtrans_max

ros2 param get /ratslam_pose_cells pc_vt_inject_energy
ros2 param get /ratslam_pose_cells exp_delta_pc_threshold

ros2 param get /ratslam_experience_map exp_loops

ros2 topic info /clock
ros2 topic info /stlucia/camera/image/compressed
ros2 topic info /stlucia/odom
ros2 topic hz /clock
ros2 topic hz /stlucia/camera/image/compressed
ros2 topic hz /stlucia/odom
ros2 topic echo /stlucia/camera/image/compressed --once
ros2 topic echo /stlucia/odom --once
ros2 topic echo /stlucia/view_template --once
ros2 topic echo /stlucia/PoseCell/TopologicalAction --once
```

---

## 12. 빠른 체크리스트

아래 항목이 맞으면 현재 상태는 최소한 **논문과 크게 어긋나지 않는 정상 범위**로 볼 수 있다.

### 파라미터

- `vt_match_threshold = 0.085`
- `template_x_size = 60`
- `template_y_size = 10`
- `vt_shift_match = 5`
- `vt_step_match = 1`
- `vt_panoramic = 0`
- `vt_normalisation = 0.4`
- `camera_hz = 10`
- `camera_fov_deg = 53`
- `vtrans_scaling = 1000`
- `vtrans_max = 20`
- `pc_vt_inject_energy = 0.15`
- `exp_delta_pc_threshold = 1.0` on pose cells
- `exp_loops = 20`

### 토픽

- bag가 `/stlucia/camera/image/compressed` publish
- VO가 `/stlucia/odom` publish
- Pose Cells, Experience Map이 `/stlucia/odom` subscribe
- bag play 중 `/clock` publisher 수가 1

### odom

- `/stlucia/odom` Hz가 이미지 Hz와 크게 다르지 않음
- 현재 샘플은 `image ≈ 10 Hz`, `odom ≈ 10 Hz`
- `linear.x`, `angular.z`가 0만 나오지 않음

### Local View 해석

- `Templates ≈ Iteration` 자체는 실패 조건이 아님
- **실제로 봐야 하는 것은 `VTM[...]`가 반복적으로 나타나는지** 여부
- 현재 샘플에서는 `VTM > VTN` 이므로 재사용이 존재함

### 논문과 가까운 최소 조건

- full bag 전체 실행
- `/clock` 정상
- image / odom Hz 안정
- YAML / runtime 값이 논문값과 일치
- `VTM` 재사용 구간 존재
- **experience 증가 속도 > template 증가 속도** 경향 유지

---

## 13. 이번 점검에서 수정한 파일

이번 St Lucia 재현성 점검 단계에서는 **소스 / launch / config를 추가 수정하지 않았다.**

생성된 것은 런타임 로그뿐이다.

- `launch.log`: St Lucia launch 20초 샘플 로그
- `bag.log`: St Lucia bag play 로그

즉,

- **코어 알고리즘 변경 없음**
- **config/launch 변경 없음**
- **이번 단계는 런타임 검증 및 해석 정리 단계**

---

## 14. 나중에 다시 볼 때 가장 먼저 확인할 5줄

1. **`Templates ≈ Iteration` 은 실패 증거가 아니다.**
2. **실제로 봐야 하는 것은 `VTM` 재사용이 반복되는지다.**
3. **현재 샘플에서는 `VTM > VTN` 이므로 Local View 재사용은 존재한다.**
4. **핵심 파라미터 / 토픽 / sim time / odom은 논문과 대체로 정합적이다.**
5. **현재 근거만으로는 Humble 포팅본이 비정상이라고 단정하기 어렵다.**
