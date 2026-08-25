# 실험 계획

## 방향

1. **벤치마크 하나 선정**

2. **Upper bound 실험**
   VoLoAgent를 사용해서, 한 episode에서 얻을 수 있는 최대한 많은 정보를 주고 reflection했을 때 성능이 오르는지 확인.
   → "reflection이 효과가 있다"는 것을 먼저 입증.

3. **Ablation으로 method design**
   정보를 하나씩 제거해가면서 어떤 정보가 중요한지 파악.
   → 실제로 필요한 최소한의 정보 조합을 찾아 method를 설계.
