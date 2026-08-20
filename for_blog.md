# 연구 진행 기록 (블로그용)

---

## 2026-08-20

### 1. CaP-X Docker
HuggingFace에서 직접 다운 시도했으나 막혀서 상혁이한테 부탁함.

### 2. 연구소 VLM Serving
SmolVLM2 제외하고 전부 서빙 성공.

**성공한 모델:**
- Qwen2.5-VL-3B-Instruct
- Qwen2.5-VL-7B-Instruct
- Qwen2.5-VL-32B-Instruct
- InternVL2_5-8B
- InternVL2_5-38B
- DeepSeek-VL2 (27B MoE, `--hf-overrides '{"architectures": ["DeepseekVLV2ForCausalLM"]}'` 필요)

**성공 (연구소 내 추가):**
- Phi-4-multimodal-instruct (audio 전용 모듈 trick으로 우회해서 해결)

**실패한 모델:**
- SmolVLM2-2.2B-Instruct (`num2words` 패키지 없음)

서빙 커맨드: A 머신(192.168.0.5) `/raid/seokmin_agent/cap-x_main/vlms/vlm_turn_on.md` 참고.
쿼리: D 머신(4090)에서 `vlm_query.sh` 이용.

### 3. VoLoAgent LIBERO 실험
연구소에서 VoLoAgent + LIBERO 파이프라인 실행 성공. 단, **로봇이 이상하게 움직임 → 디버깅 필요**.

세팅 참고: D 머신 `~/seokmin/seokmin_agent/volo-main/volo-agent/seokmins/add_env_settings.md`
빠른 실행: D 머신 `~/seokmin/seokmin_agent/volo-main/volo-agent/utils/LIBERO/` 아래 sh 파일들 그대로 실행하면 됨.
