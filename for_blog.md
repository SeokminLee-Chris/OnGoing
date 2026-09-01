# 연구 진행 기록 (블로그용)

---

## 2026-09-01

### IndexError: grasp hold-chunk 크기 불일치

VLA(pi05)는 action chunk를 15개 반환하고, `open_loop_horizon=15`(기본값)는 이를 기준으로 설계됨. 그런데 grasp/place tool은 hold-position chunk를 8개만 반환하기 때문에, tool 사용 시 8개짜리 chunk를 15개 소진하려 해서 `chunk[8]` 접근 → IndexError. grasp tool이 발동되는 task에서만 재현됨. 수정: `--open-loop-horizon 8` 추가.
