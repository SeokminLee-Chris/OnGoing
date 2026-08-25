# Task 정의

## 우리의 Goal

**Episode 간 reflection을 잘 하는 것.**

이를 위해 두 가지 메모리가 필요하다:

### Step 1. Intra-episode memory (과정 기록)
한 episode가 진행되는 동안 무슨 일이 있었는지를 기록.
단순 성공/실패만으로는 "왜 실패했는지"를 알 수 없기 때문에,
의미 있는 reflection을 하려면 task 진행 과정 자체가 저장되어 있어야 함.

### Step 2. Reflection
저장된 과정을 바탕으로 실패 원인을 분석하고 교훈을 추출.

### Step 3. Inter-episode memory (교훈 저장)
Reflection으로 얻은 정보를 저장해두고, 이후 episode에서 활용해 성공률을 높임.

```
Intra-episode memory   →   Reflection   →   Inter-episode memory
  (과정 기록)               (원인 분석)         (교훈 저장 + 활용)
```
