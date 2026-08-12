# 2026-07-03 02 partial motion mesh strategy

## Context

사용자가 GUI 편집과 SIBR 렌더링 확인이 잘 된 뒤, asset 안에서도 실제로 움직일 부분은 일부이므로 그 부분만 mesh로 변환하려면 어떻게 해야 하는지 질문했다.

## Decisions

- 렌더링용 edited 3DGS asset과 시뮬레이션용 moving ROI proxy를 분리한다.
- `iteration_edit`는 전체 편집 asset을 렌더링 확인/배경 유지용으로 둔다.
- 움직일 부분만 다시 선택한 `motion ROI` PLY를 별도로 만들고, 이 파일만 mesh extraction 또는 lightweight proxy 생성 입력으로 사용한다.
- GOF `extract_mesh.py`는 `--iteration`을 integer로 받으므로, mesh extraction 입력은 `iteration_edit` 같은 문자열 폴더보다 `iteration_40001` 같은 숫자 iteration 폴더에 넣는 것이 안전하다.
- ROI mesh의 open boundary는 Wind3DGS 관점에서 자연스러운 fixed/attachment boundary 후보로 쓴다.

## Changed Files

- `ideas/sessions/2026-07-03_02_partial_motion_mesh_strategy.md`

## Verification

- `external/gaussian-opacity-fields/extract_mesh.py`가 `parser.add_argument("--iteration", default=30000, type=int)`를 사용하고, `point_cloud/iteration_<iteration>/point_cloud.ply`를 직접 읽는 것을 확인했다.
- 기존 M04 wrapper는 숫자 iteration 기반 mesh extraction을 전제로 한다.

## Next

- SuperSplat 또는 Python PLY filter로 `point_cloud_motion.ply`를 만들고, `<model>/point_cloud/iteration_40001/point_cloud.ply`에 배치한다.
- GOF mesh extraction을 `--iteration 40001`로 실행해 partial ROI mesh를 생성한다.
- 이후 전체 edited Gaussian에는 ROI membership/binding mask를 저장해, ROI 내부 Gaussian만 mesh deformation을 적용하고 바깥 Gaussian은 static/kinematic으로 유지한다.
