# 2026-06-27 03 ROI crop strategy

## Context

사용자가 이미 만든 3DGS에서 Wind3DGS 프로젝트용 관심 영역만 크롭하려면 어떻게 수정하면 되는지 질문했다. 현재 프로젝트 흐름은 GOF/3DGS 학습 산출물과 mesh extraction을 M04에서 다루고, 이후 proxy binding 및 wind deformation으로 넘기는 구조다.

## Decisions

- 가장 안전한 첫 단계는 학습된 `point_cloud.ply`에서 ROI 밖 Gaussian을 제거하는 post-filter다.
- 기존 `code/wind3dgs/m04_mesh_extraction/filter_viewer_safe_ply.py`가 binary 3DGS PLY를 읽고 outlier를 제거하므로, 여기에 `--bbox-min`, `--bbox-max`, 선택적으로 `--invert-crop` 같은 옵션을 추가하는 방식이 작고 재사용 가능하다.
- mesh extraction까지 ROI만 대상으로 하려면, 원본 model은 보존하고 `point_cloud/iteration_roi_crop/point_cloud.ply` 같은 별도 iteration/copy를 만든 뒤 viewer와 binding에서 그 PLY를 사용한다.
- 입력 이미지 자체를 2D crop해서 재학습하는 방식은 카메라 intrinsics의 principal point 보정과 COLMAP/image layout 재생성이 필요하므로, object-centric 데이터셋 정리 단계에서 별도 전처리로 다루는 것이 맞다.

## Changed Files

- `ideas/sessions/2026-06-27_03_roi_crop_strategy.md`

## Verification

- 코드 변경은 하지 않았다.
- 최근 M04 GOF 학습/밀도 제어 기록과 `filter_viewer_safe_ply.py` 구조를 확인했다.

## Next

- 실제 구현 시 `filter_viewer_safe_ply.py`에 ROI bbox 필터를 추가하고, 필요하면 `run_gof_mip360_quality_all.sh`의 SIBR 변환 command에 ROI 옵션을 전달한다.
