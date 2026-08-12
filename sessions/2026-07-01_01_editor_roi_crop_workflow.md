# 2026-07-01 01 editor ROI crop workflow

## Context

사용자가 3DGS asset에서 Maya 같은 에디터로 시뮬레이션 영역만 잘라내려면 어떻게 해야 하는지 질문했다. Wind3DGS의 현재 연구 흐름은 trained 3DGS를 입력으로 두고, 관심 영역을 proxy mesh extraction, Gaussian binding, wind simulation으로 넘기는 구조다.

## Decisions

- Maya 같은 DCC는 3DGS 원본을 직접 편집하는 도구라기보다 ROI authoring 도구로 쓰는 것이 안전하다.
- 에디터에서는 crop box, oriented box, closed mask mesh, 또는 proxy mesh face selection을 만들고, 실제 Gaussian 제거는 `point_cloud.ply` post-filter에서 수행한다.
- 최종 렌더링에서도 ROI 밖 영역이 필요하면 Gaussian을 물리적으로 삭제하지 말고, simulation ROI mask만 만들고 ROI 밖 Gaussian은 static/kinematic으로 유지한다.
- 실제 asset을 잘라낸 별도 3DGS가 필요하면 원본 model은 보존하고 `point_cloud/iteration_roi_crop/point_cloud.ply` 같은 별도 산출물로 저장한다.
- 현재 `code/wind3dgs/m04_mesh_extraction/filter_viewer_safe_ply.py`는 radius/scale/opacity 필터와 SIBR 호환 변환만 지원하므로, bbox/OBB/mesh-mask crop 옵션을 추가하는 것이 다음 구현 단계다.

## Changed Files

- `ideas/sessions/2026-07-01_01_editor_roi_crop_workflow.md`

## Verification

- `ideas/AGENTS.md`, 최근 `ideas/sessions/2026-06-27_03_roi_crop_strategy.md`, `ideas/idea_sketch.tex`, `code/wind3dgs/m04_mesh_extraction/filter_viewer_safe_ply.py` 구조를 확인했다.
- `../RESEARCH_PROJECT_GUIDE.md`와 `../templates/research_project/TEMPLATE_MANIFEST.md`는 현재 workspace 기준 안내된 상대 경로에서 발견되지 않았다.

## Next

- `filter_viewer_safe_ply.py`에 `--bbox-min`, `--bbox-max`, 추후 `--obb-json` 또는 `--mask-mesh` 옵션을 추가한다.
- Maya/Blender/SuperSplat/CloudCompare 중 하나를 ROI authoring 도구로 정하고, 해당 도구가 내보내는 좌표계와 3DGS PLY 좌표계가 일치하는지 검증한다.
