# ROS2 Humble macOS 설치 가이드 (Apple Silicon)

> **환경** : macOS / Apple Silicon (M1/M2/M3)  
> **패키지 매니저** : Miniconda + RoboStack  
> **설치 대상** : ROS2 Humble Desktop

---

## 1. Miniconda 설치

```bash
mkdir -p ~/miniconda3
curl https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh -o ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm ~/miniconda3/miniconda.sh
```

설치 후 터미널을 재시작하거나 아래 명령어로 conda를 활성화한다.

```bash
source ~/miniconda3/etc/profile.d/conda.sh
```

---

## 2. defaults 채널 제거

Miniconda는 기본적으로 Anaconda의 `defaults` 채널이 포함되어 있다.  
이는 Anaconda ToS 위반 가능성이 있으므로 반드시 제거한다.

```bash
conda config --env --remove channels defaults
```

> [!warning]
> 이 작업은 **ros_env 환경 생성 이후** 환경이 활성화된 상태에서 실행해야 한다.  
> 환경 생성 전에는 아래 3단계를 먼저 진행한다.

---

## 3. ROS2 Humble 환경 생성 및 설치

```bash
conda create -n ros_env -c conda-forge -c robostack-humble ros-humble-desktop
```

> [!note]
> 패키지 다운로드 및 설치에 시간이 걸릴 수 있다. (네트워크 환경에 따라 수 분 소요)

---

## 4. 환경 활성화

```bash
conda activate ros_env
```

---

## 5. defaults 채널 제거 (환경 활성화 후)

```bash
conda config --env --remove channels defaults
```

---

## 6. robostack-humble 채널 등록

```bash
conda config --env --add channels robostack-humble
```

---

## 7. ROS 개발 도구 설치

```bash
conda install -c conda-forge ros-dev-tools
```

---

## 8. 설치 확인 (rviz2 실행)

```bash
rviz2
```

rviz2 창이 정상적으로 열리면 설치 완료.

---

## 주의사항

| 항목 | 내용 |
|------|------|
| ROS는 base 환경에 설치 금지 | 충돌 문제 발생 |
| conda/mamba는 ros_env에 설치 금지 | base 환경에만 존재해야 함 |
| 시스템 ROS 환경 source 금지 | PYTHONPATH 충돌 발생 |
| ~/.bashrc에 source 추가 불필요 | `conda activate` 시 자동으로 ROS 환경이 활성화됨 |

---

## 일상적인 사용법

### 환경 활성화

```bash
conda activate ros_env
```

### 환경 비활성화

```bash
conda deactivate
```

### 패키지 전체 업데이트

```bash
conda activate ros_env
conda update --all
```

---

## 참고

- [RoboStack 공식 문서](https://robostack.github.io/GettingStarted.html)
- [Miniforge (권장 설치 방법)](https://github.com/conda-forge/miniforge)


## 9. 추가 패키지 설치

RoboStack 환경에서는 `apt` 대신 `conda`로 ROS 패키지를 설치한다.

```bash
conda install -c robostack-humble ros-humble-xacro ros-humble-joint-state-publisher-gui
````

### apt vs conda 비교

||Ubuntu (apt)|RoboStack (conda)|
|---|---|---|
|설치|`sudo apt install ros-humble-xxx`|`conda install -c robostack-humble ros-humble-xxx`|
|제거|`sudo apt remove ros-humble-xxx`|`conda remove ros-humble-xxx`|
|검색|`apt search ros-humble-xxx`|`conda search -c robostack-humble ros-humble-xxx`|
|sudo 필요|O|X|

### 패키지 존재 여부 확인

```bash
conda search -c robostack-humble ros-humble-xacro
```

또는 브라우저에서 확인 → https://prefix.dev/channels/robostack-humble

> [!note] conda에 없는 패키지는 `~/ros2_ws/src/`에 소스코드를 직접 클론한 후 `colcon build`로 빌드한다.

---

## 10. ros2_ws 워크스페이스 생성 및 빌드

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
colcon build
```

> [!warning] `src/`가 비어있는 상태에서 빌드하면 `setup.zsh` 소싱 시 오류가 발생할 수 있다.  
> 패키지를 추가한 후 빌드하고 소싱하는 것을 권장한다.

### 빌드 후 환경 소싱

```bash
# zsh 환경 (기본 macOS 터미널)
source ~/ros2_ws/install/setup.zsh
```

> [!warning] macOS의 기본 셸은 zsh이므로 `setup.bash`가 아닌 `setup.zsh`를 소싱해야 한다.

### conda activate 시 자동 소싱 설정 (선택)

매번 수동으로 소싱하는 번거로움을 없애려면 아래와 같이 설정한다.

```bash
mkdir -p ~/miniconda3/envs/ros_env/etc/conda/activate.d
echo "source ~/ros2_ws/install/setup.zsh" > ~/miniconda3/envs/ros_env/etc/conda/activate.d/ros2_ws.sh
```

이후 `conda activate ros_env` 만으로 ROS 환경과 워크스페이스가 함께 활성화된다.