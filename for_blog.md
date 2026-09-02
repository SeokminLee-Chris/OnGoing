# RoboVoLo 실험 구성 정리

> 120 tasks × 3 runs, VoLoAgent (subgoal mode), pi05_droid_jointpos
> 최종 업데이트: 2026-09-02

---

## 전체 구조

```
B300 ──────────────────────────────────────────────────────────────────
  VLM 서버 (Qwen3-27B, :13333)
        │ vlm-base-url
A100 ───┼──────────────────────────────────────────────────────────────
  openpi_N (GPU, :800N2)  ←── orch_N (CPU, :800N1)
  Molmo2 vLLM (GPU, :8122)     │ grasp/place tool
  grasp 서버  (GPU, :8003)     │
        │ remote-host/port
4090 ───┼──────────────────────────────────────────────────────────────
  robolab-eval (Isaac Sim, GPU) → orch_N :800N1
```

---

## B300 (VLM 서버)

| 역할 | 모델 | 포트 | IP |
|------|------|------|----|
| VLM 서빙 (subgoal/failure-monitor/subgoal-check) | Qwen3-27B | 13333 | 192.168.0.213 |

항상 켜두면 됨. A100 orch가 `--vlm-base-url http://192.168.0.213:13333/v1` 으로 호출.

---

## A100 터미널 구성

### parallel_4 (10 terminals)

| 터미널 | 컨테이너 | GPU | 포트 | 역할 |
|--------|----------|-----|------|------|
| T1 | openpi_1 | 0 | 8002 | Pi0.5 VLA 서버 (stream 1용) |
| T2 | openpi_2 | 1 | 8012 | Pi0.5 VLA 서버 (stream 2용) |
| T3 | molmo2_server | 1 | 8122 | Molmo2 vLLM (place segmentation) |
| T4 | grasp_server | 0 | 8003 | GraspGen + SAM3 + cuRobo |
| T5 | openpi_3 | 2 | 8022 | Pi0.5 VLA 서버 (stream 3용) |
| T6 | openpi_4 | 3 | 8032 | Pi0.5 VLA 서버 (stream 4용) |
| T7 | orch_1 | CPU | 8001 → 8002 | Orchestrator stream 1 (subgoal) |
| T8 | orch_2 | CPU | 8011 → 8012 | Orchestrator stream 2 (subgoal) |
| T9 | orch_3 | CPU | 8021 → 8022 | Orchestrator stream 3 (subgoal) |
| T10 | orch_4 | CPU | 8031 → 8032 | Orchestrator stream 4 (subgoal) |

> Molmo2 / grasp 서버는 parallel 상관없이 공유 (각 1개 인스턴스로 전 stream 담당)

---

## 4090 터미널 구성

### parallel_4 (4090 3대, 4 terminals)

| 머신 | 터미널 | GPU | 연결 대상 | tasks |
|------|--------|-----|-----------|-------|
| 4090_1 | T1 | 0 | orch_1 :8001 | 1~30번 |
| 4090_1 | T2 | 1 | orch_2 :8011 | 31~60번 |
| 4090_2 | T1 | 0 | orch_3 :8021 | 61~90번 |
| 4090_3 | T1 | 0 | orch_4 :8031 | 91~120번 |

---

## 실험 시작 순서

### 1. B300 — Qwen 서버 켜기

```bash
# B300 (192.168.0.213) 에서
bash /raid/seokmin_agent/run_qwen38_gpu0.sh
```

### 2. A100

```bash
source seokmins/exp_script/robovolo/parallel_4/a100/all_terminal_init.sh

bash terminal1.sh   # openpi_1
bash terminal2.sh   # openpi_2
bash terminal3.sh   # molmo2_server
bash terminal4.sh   # grasp_server
bash terminal5.sh   # openpi_3
bash terminal6.sh   # openpi_4
bash terminal7.sh   # orch_1
bash terminal8.sh   # orch_2
bash terminal9.sh   # orch_3
bash terminal10.sh  # orch_4
```

### 3. 4090 (각 머신에서)

```bash
# 4090_1
source seokmins/exp_script/robovolo/parallel_4/4090_1/all_terminal_init.sh
bash terminal1.sh   # tasks 1~30
bash terminal2.sh   # tasks 31~60

# 4090_2
source seokmins/exp_script/robovolo/parallel_4/4090_2/all_terminal_init.sh
bash terminal1.sh   # tasks 61~90

# 4090_3
source seokmins/exp_script/robovolo/parallel_4/4090_3/all_terminal_init.sh
bash terminal1.sh   # tasks 91~120
```

---

## 로그 위치 및 내용

### A100 — `/tmp/debug*` (orch debug log)

orch_N마다 `/tmp/debugN`에 저장됨 (컨테이너 내 `/logs/debug` → 호스트 `/tmp/debugN` 마운트).

```
/tmp/debug1/                        ← orch_1 (stream 1)
/tmp/debug2/                        ← orch_2 (stream 2)
/tmp/debug3/                        ← orch_3 (stream 3)
/tmp/debug4/                        ← orch_4 (stream 4)
    ep_001_pick_up_the_banana/
        step.jsonl                  ← step별 전체 이벤트 로그 (한 줄 = 1 JSON)
        vlm/
            s0000_decompose.jpg     ← VLM 호출 시 보낸 이미지
            s0050_check.jpg         ← subgoal check 시 이미지
        grasp/
            s0080_grasp.jpg         ← grasp 호출 시 이미지
    ep_002_.../
        ...
```

`step.jsonl` 에 기록되는 이벤트:

| 이벤트 | 내용 |
|--------|------|
| `obs_received` | 매 step obs keys & shapes |
| `vlm_call` | VLM instruction / raw response / parsed / latency / fallback 여부 |
| `subgoal_list` | decompose 결과 subgoal 전체 목록 |
| `subgoal_state` | 현재 subgoal idx/text/진행 steps / advance·timeout 이벤트 |
| `failure_detection` | failure type / confidence / recovery action / reason |
| `grasp_call` | grasp target / pose / success·fail / latency |
| `action_routing` | action 출처 (vla / grasp_tool / place_tool) + instruction + joint 요약 |
| `gt_state` | EE pos / gripper / grasped object / 물체별 z_lift |
| `timing` | VLM·grasp 등 component별 latency |
| `step_wall` | orchestrator 전체 step wall time |

---

### 4090 — `$VIDEOS/p4_stack*` (eval 결과)

4090 eval 결과는 각 머신의 `$VIDEOS` 하위에 저장됨.

```
~/seokmin/seokmin_agent/videos/
    p4_stack1/                      ← 4090_1 GPU0 (tasks 1~30)
    p4_stack2/                      ← 4090_1 GPU1 (tasks 31~60)
    p4_stack3/                      ← 4090_2 GPU0 (tasks 61~90)
    p4_stack4/                      ← 4090_3 GPU0 (tasks 91~120)
        <timestamp>_pi05_vague/
            results.json            ← 전체 episode 성공/실패 결과
            timing.json             ← task별 episode 시간 통계
            <TaskName>/
                <instruction>_ep0.mp4   ← sensor 카메라 영상 (run 1)
                <instruction>_ep1.mp4   ← sensor 카메라 영상 (run 2)
                <instruction>_ep2.mp4   ← sensor 카메라 영상 (run 3)
```

---

## TODO: 실험 결과 나오면 분석하기

1. **Time 분석** — task별로 얼마나 걸리는지 (episode 시간, 구간별 병목 파악)
2. **실패 원인 분석** — 왜 실패하는지 (grasp 실패? subgoal 오판? VLA 동작 이상?)
3. **추가 디버깅 필요 여부 분석** — debug log 보고 추가로 손봐야 할 게 있는지
