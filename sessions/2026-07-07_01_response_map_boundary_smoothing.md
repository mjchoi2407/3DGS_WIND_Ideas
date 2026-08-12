# 2026-07-07 01 response map boundary smoothing

## Context

사용자가 wind response를 적용할 ROI crop 이후, 나무처럼 줄기는 단단하고 잎은 부드럽게 움직이는 부위별 강도를 어떻게 설정할지 물었다. 이어서 부위 경계에서 motion discontinuity가 생기지 않게 하는 방법을 아이디어 스케치에 반영해달라고 요청했다. 또한 기존 연구인 Gaussian Swaying이 부위별 강도 제어 파라미터를 제공하는지 확인해달라고 했다.

## Gaussian Swaying 확인

확인한 자료:

- arXiv: `Gaussian Swaying: Surface-Based Framework for Aerodynamic Simulation with 3D Gaussians`, arXiv:2512.01306
- PDF를 `/tmp/gaussian_swaying.pdf`로 내려받아 `pdftotext`로 본문을 검색했다.

확인 내용:

- Gaussian Swaying은 Gaussian을 surface patch로 정의하고 각 patch에 area, normal, opacity, color를 둔다.
- 공력은 drag, friction, lift 조합으로 계산하며 \(C_D,C_F,C_L\) 계수를 사용한다.
- implementation detail에는 simulation region, object material, parameter를 수동 지정한다고 되어 있다.
- 보충 표에는 scene 단위 Young's modulus, Poisson ratio, density, \(C_D,C_F,C_L\), flow intensity, constitutive model이 제시되어 있다.
- 내가 확인한 범위에서는 줄기/잎/가지처럼 사용자가 부위별 response strength를 칠하고, 경계를 연속값으로 smoothing하는 전용 control layer는 핵심 장치로 보이지 않았다.

## Decisions

- ROI crop 결과와 부위 label은 runtime deformation에 직접 쓰는 hard mask가 아니라 seed로만 사용한다.
- 실제 simulation에는 \(0\)--\(1\) 연속 response map을 사용한다.
- crop 경계에는 smoothstep feather band를 둔다.
- semantic boundary에는 constrained Laplacian smoothing을 적용한다.
- trunk/branch/leaf 같은 내부 seed는 confidence를 높게 유지하고, 경계 주변만 부드럽게 보간한다.
- Gaussian 단위 response는 bound triangle의 vertex response를 barycentric interpolation하여 상속한다.

## Changed Files

- `idea_sketch.tex`
  - `kotex` package 추가.
  - Gaussian Swaying 확인 내용을 related-work boundary로 추가.
  - `부위별 Wind Response Map과 경계 연속화` subsection 추가.
  - contribution, ablation, risk, decision log에 response map 내용을 추가.
- `changelog.md`
  - 2026-07-07 response map decision 추가.
- `sessions/2026-07-07_01_response_map_boundary_smoothing.md`
  - 본 세션 기록 추가.

## Verification

다음 명령으로 `idea_sketch.pdf` 빌드를 확인했다.

```bash
latexmk -xelatex -interaction=nonstopmode -halt-on-error idea_sketch.tex
```

빌드는 성공했다. 한국어 문장을 위해 `kotex` package를 추가했고, XeLaTeX에서 PDF가 생성되는 것을 확인했다. 빌드 로그에는 일부 한글 italic font shape 대체 warning이 있었지만, 컴파일 실패는 없었다.

## Next

- `idea_sketch.pdf` 빌드를 확인한다.
- ROI crop/control prototype에서는 hard label과 smoothed response map을 모두 저장한다.
- 첫 UI는 seed brush, crop feather width, smoothing strength \(\lambda_E\), response preset(trunk/branch/leaf)을 제공하는 방향이 적합하다.
