# 2026-07-03 01 GUI ROI crop tools

## Context

사용자가 3DGS에서 시뮬레이션 영역을 자르기 위한 GUI 기반 편집 프로그램이 있는지 질문했다. 이전 결정은 Maya 같은 DCC를 원본 3DGS 편집 도구가 아니라 ROI authoring 도구로 쓰고, 실제 crop은 3DGS PLY post-filter에서 수행하는 방향이었다.

## Decisions

- 현재 목적에는 SuperSplat이 가장 직접적인 GUI 후보로 보인다.
- SuperSplat은 browser 기반 3D Gaussian Splat editor이며, 공식 문서상 view, inspect, transform, combine, crop, clean up, optimize를 지원한다.
- SuperSplat에서 직접 splat selection/delete/save로 cropped PLY를 만들 수 있지만, Wind3DGS 연구 파이프라인에서는 원본 보존과 재현성을 위해 GUI crop 결과를 검증하거나 동일 ROI를 Python post-filter로 재현하는 편이 안전하다.
- CloudCompare/MeshLab 같은 일반 point cloud/mesh editor는 ROI를 눈으로 잡는 데는 쓸 수 있으나, 3DGS 전용 속성 보존과 렌더링 호환성은 보장되지 않으므로 최종 3DGS 산출물 편집 도구로는 비추천한다.
- Maya/Blender는 proxy mesh, crop box, mask mesh authoring에는 좋지만 원본 3DGS PLY를 직접 잘라 저장하는 주 도구로 쓰기에는 위험하다.

## Changed Files

- `ideas/sessions/2026-07-03_01_gui_roi_crop_tools.md`

## Verification

- SuperSplat 공식 site/GitHub/user guide를 확인했다.
- MeshLab 공식 site와 CloudCompare wiki의 point cloud/mesh segmentation 성격을 확인했다.

## Next

- 실제 crop 작업은 SuperSplat에서 빠르게 시각 검토한 뒤, 연구 산출물로는 `filter_viewer_safe_ply.py`에 bbox/OBB/mask crop을 추가해 재현 가능한 PLY를 생성하는 흐름이 적합하다.
