# 2026-08-13 03 global/local 물리 반응 파이프라인

## 핵심 설계

Runtime 물리 상태를 `global object state`와 `active local residual state`로 나눈다. 두 레벨 모두 최종 위치를 직접 회귀하지 않고, 힘 또는 가속도를 구한 뒤 시간 적분으로 상태를 갱신한다.

- global: 전체 물체의 저주파 굽힘, 흔들림, 관성, 복원, 감쇠를 항상 계산한다.
- local: global basis가 표현하지 못한 flutter, fold, 국소 비선형 반응만 선택된 patch에서 계산한다.
- local correction은 시각적 후처리가 아니라 reaction force와 수정된 aerodynamic geometry를 통해 다음 global step에 되먹임한다.

## Offline object package

Training-only mesh 또는 고해상도 teacher simulation에서 다음 정보를 static 3DGS용 runtime package로 distill한다.

- oriented thin-surface anchor graph와 Gaussian binding
- anchor별 면적, 질량, rest normal, material, attachment
- object-wide global deformation modes와 reduced mass/stiffness/damping
- patch별 local complement basis와 local mass/stiffness/damping
- global approximation error estimator와 local missing-force predictor
- aerodynamic generalized-force용 conservative sample set

Target runtime에서는 teacher mesh나 MPM grid를 사용하지 않는다.

## Runtime global pipeline

1. 이전 global deformation/velocity와 이전 local correction으로 현재 anchor의 위치, 속도, normal, area를 구성한다.
2. spatial wind에서 각 aerodynamic sample 위치의 풍속을 읽고, 표면 속도를 빼 실제 상대풍을 만든다.
3. 상대풍, current normal/area, material coefficient를 공기력 식에 넣어 sample별 force를 구한다.
4. sample force를 object-wide deformation modes가 받는 generalized force로 보존적으로 집계한다.
5. 이전 modal state, generalized wind force, gravity/attachments, mass/stiffness/damping을 reduced structural integrator에 넣는다.
6. 새 modal displacement/velocity를 얻고 모든 anchor와 Gaussian으로 lift하여 global predictor geometry를 만든다.

Structural restoring force와 damping은 local gate와 무관하게 항상 켜 둔다.

## Runtime local pipeline

1. global predictor의 patch별 strain, curvature, acceleration, local wind, mode truncation indicator, 과거 error를 error estimator에 넣는다.
2. posterior error/uncertainty가 큰 patch만 active set으로 고른다. hysteresis를 사용해 on/off 경계의 깜빡임을 막는다.
3. active patch의 global boundary motion, local current state, wind, material을 missing-force predictor에 넣어 global solver가 놓친 local corrective force를 구한다.
4. patch별 작은 residual mass/stiffness/damping system을 적분하여 local displacement와 velocity를 갱신한다. 네트워크가 최종 위치를 직접 출력하지 않는다.
5. residual은 mass-orthogonal complement에 제한해 global modes와 운동을 중복하지 않게 한다.
6. overlapping patch를 합칠 때 net force/impulse와 torque를 보존하고, 얻은 reaction force를 global modal force로 돌려보낸다.
7. global predictor와 local residual을 합친 corrected geometry에서 normal, area, surface velocity와 aerodynamic force를 다시 갱신한다.

Inactive patch는 learned residual evaluation을 생략한다. 아직 local energy가 남은 patch는 저렴한 선형 감쇠 적분만 계속하고, 에너지가 충분히 작아진 뒤 상태를 종료한다.

## Frame 단위 연결

```text
previous global/local states + wind
-> global aerodynamic/structural predictor
-> patch error estimation
-> active local missing-force solve and integration
-> complementary and conservative assembly
-> reaction force + corrected aerodynamic geometry
-> optional global corrector or next-frame feedback
-> Gaussian mean/covariance transport and rendering
```

안정성이 필요한 경우 한 frame 안에서 `global predictor -> local corrector -> global correction`을 한 번 수행한다. 가장 빠른 구성은 local reaction을 다음 frame의 global force에 반영한다.

## Teacher label 구성

고해상도 MPM/FEM teacher trajectory를 global basis에 먼저 투영한다. global reconstruction으로 설명되는 부분을 제거한 나머지를 local target으로 사용한다.

- gate target: global-only 결과의 patch별 미래 오차 또는 teacher residual energy
- local target: teacher와 global model 사이의 missing force/acceleration
- feedback target: local correction을 포함했을 때의 next global generalized force/state

이 분해로 structural base를 residual label에 섞거나 global/local이 같은 운동을 두 번 예측하는 것을 막는다.
