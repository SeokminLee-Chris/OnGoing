# 벤치마크 검증 현황

> 작성: 2026-08-17

---

## CaP-X

### 환경별 검증

| 벤치마크 | 환경 | 검증 내용 | 상태 |
|---------|------|---------|------|
| **Robosuite** (FrankaPickPlace) | Robosuite 1.5 | FrankaPickPlaceCodeEnv reset + oracle code 실행 (LLM 없이 5 steps) | ✅ PASS |
| **LIBERO-PRO** | MuJoCo/robosuite 1.4 | `load_libero_task()` + `env.reset()` (libero_spatial / libero_object / libero_goal) | ✅ PASS |
| **BEHAVIOR-1K** (OmniGibson) | Isaac Sim 4.5 | `og.launch()` + `R1ProBehaviourLowLevel(picking_up_trash)` + `load_task_instance(0)` + `reset()` → obs PASS | ✅ PASS (마운트 필요) |
| 지각 서버 (SAM3 / GraspNet / PyRoKi) | — | 실 GPU 서버 기동 + API 응답 (200 masks, 102 grasps, IK) | ✅ PASS |
| **LLM 연동 실제 trial** | 전체 파이프라인 | vLLM → CaP-X → 시뮬레이터 end-to-end | ❌ 미검증 (연구소에서 첫 실행) |

### 비고

- Robosuite/LIBERO는 **환경 초기화 수준** 검증. LLM 붙인 실제 trial은 연구소에서 첫 실행
- BEHAVIOR-1K는 씬 로딩 + task 초기화 + reset() 전체 검증 완료 (~25분 소요: Isaac Sim 기동 3분 + 씬 로딩 8분 + CuRoBo JIT 5분 + reset 1분)
- OmniGibson 실행 시 **마운트 4종 필요** (상세: `cap-x_main/cap-x/seokmins/add_env_settings.md` 섹션 6-5)
  - `/usr/lib/x86_64-linux-gnu` (libSM.so.6)
  - `/usr/share/vulkan/icd.d` (nvidia_icd.json)
  - `/tmp/.X11-unix` (Xvfb 소켓)
  - `mount/omnigibson_4_5_0.kit` (XR 확장 제거)

---

## VoLoAgent

### 환경별 검증

| 벤치마크 | 모드 | 검증 내용 | 상태 |
|---------|------|---------|------|
| **LIBERO** (libero_10) | passthrough | 10 tasks × 3 trials end-to-end | ✅ PASS |
| **VLABench** (10 tasks) | passthrough | 10 tasks × 1ep, 70% 성공률 | ✅ PASS |
| Grasp 서버 | — | latency × 3회 / Orchestrator 동시 5 client | ✅ PASS |
| **LIBERO** (libero_goal) | passthrough | 10 tasks × 1 trial | ✅ PASS |
| **LIBERO** (libero_spatial) | passthrough | 10 tasks × 1 trial | ✅ PASS |
| **LIBERO** (libero_object) | passthrough | 10 tasks × 1 trial | ✅ PASS |
| **LIBERO** (libero_90) | passthrough | 1 task (파이프라인 확인) | ✅ PASS |
| **subgoal 모드** (VLM 연동) | subgoal | VLM → subgoal 생성 → VLA | ❌ 미검증 (GPU OOM, 연구소에서) |
| **RoboCasa** | passthrough | — | ❌ 미검증 (Docker 이미지 빌드 필요) |
| **droid** (Isaac Sim / IsaacLab) | passthrough | 3 scenes (cube/can/banana) | ❌ 미검증 (Docker 이미지 빌드 필요) |
| **RoboVoLo** (RoboLab / Isaac Sim) | subgoal | 126 tasks, 123 USD scenes | ❌ 미검증 (Docker 이미지 빌드 + SimReady 라이선스) |

### 비고

- passthrough 모드 (VLM 없이 VLA 직접 호출) 는 풀 파이프라인 검증 완료
- subgoal 모드 미검증 이유: 7B VLM(15.67GB) + SAM3/GraspNet 동시 올리면 4090 24GB OOM
  → 연구소 D 머신 GPU 1번에 별도 vLLM 서버 올리면 해결 가능
- validate_v5c.sh 기준 **16/16 PASS** (2026-08-17)
- LIBERO 전 suite 추가 검증 완료 (2026-08-17): libero_spatial/object/90 PASS
- `libero_100` 은 존재하지 않는 이름 — 올바른 이름은 `libero_90`

### 미검증 항목 상세

| 항목 | 스택 | Docker 이미지 | 추가 작업 |
|------|------|-------------|---------|
| **RoboCasa** | MuJoCo / robosuite | ❌ 없음 | robocasa 패키지 포함 이미지 빌드 |
| **droid** | Isaac Sim / IsaacLab | ❌ 없음 | Isaac Sim + IsaacLab + sim-evals 이미지 빌드, assets 다운로드 |
| **RoboVoLo** | RoboLab / Isaac Sim | ❌ 없음 | Isaac Sim + RoboLab + SimReady assets (NVIDIA 라이선스 동의 필요) |
| **subgoal 모드** | — | ✅ 기존 이미지 | 연구소 GPU 분리 (D GPU 1번에 vLLM 서버) |

> **참고:** CaP-X BEHAVIOR-1K(OmniGibson)는 capx:latest 이미지(69.3GB)에 Isaac Sim이 이미 구워져 있어서 마운트만으로 됐음.
> droid/RoboVoLo는 Docker 이미지 자체가 없어서 Isaac Sim 기반 이미지를 새로 빌드해야 하는 차이.
>
> **RoboVoLo** (https://github.com/NVlabs/RoboVoLo):
> - 설치: RoboLab v0.3.0 → RoboVoLo clone → SimReady assets 다운로드(라이선스 동의) → install.py
> - droid 체크포인트(pi05_droid_jointpos) 사용
