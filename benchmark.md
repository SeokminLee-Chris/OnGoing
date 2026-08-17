# 벤치마크 검증 현황

> 작성: 2026-08-17

---

## CaP-X

### 환경별 검증

| 벤치마크 | 환경 | 검증 내용 | 상태 |
|---------|------|---------|------|
| **Robosuite** (FrankaPickPlace 등 8종) | Robosuite 1.5 | FrankaPickPlaceCodeEnv reset + oracle code 실행 | ✅ PASS |
| **LIBERO-PRO** | MuJoCo/robosuite 1.4 | `load_libero_task()` + `env.reset()` (spatial/object/goal) | ✅ PASS |
| **BEHAVIOR-1K** (OmniGibson) | Isaac Sim 4.5 | `og.launch()` + `R1ProBehaviourLowLevel` + `load_task_instance(0)` + `reset()` → obs PASS | ✅ PASS (마운트 필요) |
| 지각 서버 (SAM3 / GraspNet / PyRoKi) | — | 실 GPU 서버 기동 + API 응답 (200 masks, 102 grasps, IK) | ✅ PASS |
| **LLM 연동 실제 trial** | 전체 파이프라인 | vLLM → CaP-X → 시뮬레이터 end-to-end | ❌ 미검증 (연구소에서 첫 실행) |

### 비고

- Robosuite 8종 (cube_lifting / cube_restack / cube_stack / nut_assembly / spill_wipe / two_arm_handover / two_arm_lift / franka_pick_place) 모두 capx:latest 이미지로 지원, YAML config만 바꾸면 됨
- BEHAVIOR-1K는 씬 로딩 + task 초기화 + reset() 전체 검증 완료 (~25분 소요)
- OmniGibson 실행 시 **마운트 4종 필요** (상세: `cap-x_main/cap-x/seokmins/add_env_settings.md` 섹션 6-5)
- Real robot (Franka Panda + ZED 카메라) 지원 코드 있으나 물리 장비 필요 → 검증 범위 외

---

## VoLoAgent

### 환경별 검증

| 벤치마크 | 모드 | 검증 내용 | 상태 |
|---------|------|---------|------|
| **LIBERO** (libero_10) | passthrough | 10 tasks × 3 trials end-to-end | ✅ PASS |
| **LIBERO** (libero_goal) | passthrough | 10 tasks × 1 trial | ✅ PASS |
| **LIBERO** (libero_spatial) | passthrough | 10 tasks × 1 trial | ✅ PASS |
| **LIBERO** (libero_object) | passthrough | 10 tasks × 1 trial | ✅ PASS |
| **LIBERO** (libero_90) | passthrough | 1 task (파이프라인 확인) | ✅ PASS |
| **VLABench** (10 tasks) | passthrough | 10 tasks × 1ep, 70% 성공률 | ✅ PASS |
| Grasp 서버 | — | latency × 3회 / Orchestrator 동시 5 client | ✅ PASS |
| **subgoal 모드** (VLM 연동) | subgoal | VLM → subgoal 생성 → VLA | ❌ 미검증 (GPU OOM, 연구소에서) |
| **LIBERO-Plus** | passthrough | 4 suites × 7 perturbation dimensions × 5 levels | ❌ 미검증 (별도 Docker 이미지 필요) |
| **LIBERO-PRO** | passthrough | 4 suites × distractor object variants | ❌ 미검증 (별도 Docker 이미지 필요) |
| **LIBERO-Mem** | passthrough/subgoal | 10 memory tasks, subgoal completion rate | ❌ 미검증 (별도 Docker 이미지 필요) |
| **RoboCerebra** | passthrough/subgoal | 5 task types (NeurIPS 2025) | ❌ 미검증 (별도 Docker 이미지 필요) |
| **RoboCasa** | passthrough | 365 tasks (18 atomic + composite) | ❌ 미검증 (별도 Docker 이미지 필요) |
| **droid** (Isaac Sim / IsaacLab) | passthrough | 3 scenes (cube/can/banana) | ❌ 미검증 (Isaac Sim 환경 필요) |
| **RoboVoLo** (RoboLab / Isaac Sim) | subgoal | 126 tasks, 123 USD scenes | ❌ 미검증 (Isaac Sim 환경 필요) |

### 미검증 항목 상세

#### MuJoCo 계열 (Docker 이미지 빌드로 해결 가능)

| 항목 | 추가 필요 사항 |
|------|-------------|
| **LIBERO-Plus** | LIBERO-plus 패키지(libero 교체) + HuggingFace assets (Sylvest/LIBERO-plus) + 별도 venv |
| **LIBERO-PRO** | libero-pro 패키지(libero 교체) + distractor object assets |
| **LIBERO-Mem** | LIBERO-Mem 패키지(libero 교체) + 별도 venv |
| **RoboCerebra** | RoboCerebra repo (qiuboxiang/RoboCerebra) + HuggingFace dataset (qiukingballball/RoboCerebraBench) |
| **RoboCasa** | robocasa 패키지 + PandaMobile assets |

> **참고:** LIBERO-Plus/PRO/Mem은 표준 libero 패키지를 교체하는 방식이라
> 서로 다른 venv가 필요하고, 기존 libero-eval Docker 이미지와 별도 이미지를 빌드해야 함.

#### Isaac Sim 계열 (환경 구성 필요)

| 항목 | 추가 필요 사항 |
|------|-------------|
| **droid** | Isaac Sim + IsaacLab + sim-evals venv + assets |
| **RoboVoLo** | RoboLab v0.3.0 + SimReady assets (NVIDIA 라이선스 동의) + install.py |

> **참고:** CaP-X BEHAVIOR-1K(OmniGibson)는 capx:latest(69.3GB)에 Isaac Sim이 이미 구워져 있어서
> 마운트만으로 됐음. droid/RoboVoLo는 Docker 이미지 자체가 없어서 새로 빌드해야 함.

#### 연구소 GPU 분리 필요

| 항목 | 이유 |
|------|------|
| **subgoal 모드** | 7B VLM(~15GB) + VLA(~15GB) = 4090 24GB OOM → GPU 분리 필요 |

### validate_v5c.sh 기준 **16/16 PASS** (2026-08-17)

- LIBERO 전 suite 추가 검증 완료 (2026-08-17): libero_goal/spatial/object/90 PASS
- `libero_100` 은 존재하지 않는 이름 — 올바른 이름은 `libero_90`
- **RoboVoLo** (https://github.com/NVlabs/RoboVoLo):
  설치: RoboLab v0.3.0 → RoboVoLo clone → SimReady assets 다운로드 → install.py
