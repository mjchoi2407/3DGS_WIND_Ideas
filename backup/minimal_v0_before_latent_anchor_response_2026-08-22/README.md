# 2026-08-22 latent response pivot 이전 Minimal V0

이 디렉터리는 2026-08-22 연구 방향 전환 직전의 canonical 문서와 파생 개념 가이드를
원래 파일명으로 보존한다. 당시 방법은 static 3DGS에서 explicit sparse physical scaffold를
증류하고, fixed-Hessian 구조 연산자와 Global/complementary Local basis를 구성해 선택적으로
적분하는 방향이었다.

새 방향은 training-only mesh simulation을 privileged teacher로 사용하되, target inference에서
mesh나 persistent physical graph를 만들지 않고 static 3DGS로부터 latent response package를
예측한다. 따라서 이 archive의 수식, checklist 완료 상태와 개념 가이드는 새 방법의 구현 요구나
완료 근거로 승계하지 않는다.

## 보존 파일

- 이전 canonical sketch TeX/PDF/bundle
- 이전 canonical bibliography
- 이전 implementation checklist TeX/PDF/bundle
- 이전 비권위 concept guide Markdown/PDF

`SHA256SUMS`는 이동 직전 기록과 일치하는지 검증하기 위한 manifest다.
