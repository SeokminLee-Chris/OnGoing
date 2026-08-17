# 벤치마크 검증 현황

> 작성: 2026-08-17

---

## CaP-X

### 환경별 검증

| 벤치마크 | 환경 | 검증 내용 | 상태 |
|---------|------|---------|------|
| **Robosuite** (FrankaPickPlace) | Robosuite 1.5 | FrankaPickPlaceCodeEnv reset + oracle code 실행 (LLM 없이 5 steps) | ✅ PASS |
| **LIBERO-PRO** | MuJoCo/robosuite 1.4 | `load_libero_task()` + `env.reset()` (libero_spatial / libero_object / libero_goal) | ✅ PASS |
| **BEHAVIOR-1K** (OmniGibson) | Isaac Sim 4.5 | `og.launch()` → Simulation App Startup Complete + 정상 종료 | ✅ PASS (마운트 필요) |
| 지각 서버 (SAM3 / GraspNet / PyRoKi) | — | 실 GPU 서버 기동 + API 응답 (200 masks, 102 grasps, IK) | ✅ PASS |
| **LLM 연동 실제 trial** | 전체 파이프라인 | vLLM → CaP-X → 시뮬레이터 end-to-end | ❌ 미검증 |
| **BEHAVIOR-1K 씬 로딩** | Isaac Sim | 실제 task 로딩 (og.launch() 이후 단계) | ❌ 미검증 |

### 비고

- Robosuite/LIBERO는 **환경 초기화 수준** 검증. LLM 붙인 실제 trial은 연구소에서 첫 실행
- BEHAVIOR-1K는 시뮬레이터 기동까지만 확인. 씬/task 로딩은 연구소에서 확인 필요
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
| **LIBERO** (libero_goal / spatial / object / 100) | passthrough | — | ❌ 미검증 (코드 지원됨) |
| **subgoal 모드** (VLM 연동) | subgoal | VLM → subgoal 생성 → VLA | ❌ 미검증 (GPU OOM 이슈) |
| **droid** checkpoint | passthrough | — | ❌ 미검증 |

### 비고

- passthrough 모드 (VLM 없이 VLA 직접 호출) 는 풀 파이프라인 검증 완료
- subgoal 모드 미검증 이유: 7B VLM(15.67GB) + SAM3/GraspNet 동시 올리면 4090 24GB OOM
  → 연구소 D 머신 GPU 1번에 별도 vLLM 서버 올리면 해결 가능
- validate_v5c.sh 기준 **16/16 PASS** (2026-08-17)
