# 2026-08-21 로컬 작업공간 정리 백업

이 디렉터리는 `ideas/` 최상위를 정리하면서 옮긴 재생성 가능 파일과 로컬 메타데이터의 복구용
백업이다. 현행 연구 문서, 구현 입력 또는 실험 증거가 아니며, `.gitignore`에서 이 README를 제외한
내용 전체를 로컬 전용으로 둔다.

## 구성

- `build_artifacts/full_design_2026-08-13/`: 구 full-design sketch/checklist LaTeX 중간산출물
- `build_artifacts/minimal_v0_2026-08-15/`: 당시 현행이었던 이전 Minimal V0 sketch/checklist의 LaTeX 중간산출물
- `build_artifacts/latent_anchor_response_2026-08-22/`: latent-response 최초 draft build의 재생성 가능한 중간산출물
- `build_artifacts/latent_anchor_response_2026-08-22_final/`: contract-closure 감사 직후 build 중간산출물
- `build_artifacts/latent_anchor_response_2026-08-22_release/`: 최종 release PDF를 만든 build 중간산출물
- `download_metadata_sidecars/`: 리뷰 파일과 함께 내려온 `Zone.Identifier` sidecar
- `guide_pdf_build_source/`: 개념 가이드 PDF를 만들 때 사용한 파생 XeLaTeX wrapper
- `SHA256SUMS`: 이동한 로컬 파일의 무결성 목록

`Zone.Identifier`에는 다운로드 출처가 포함될 수 있으므로 공유·commit하지 않는다. LaTeX
중간산출물은 canonical source로 다시 생성할 수 있으며, 복원이 필요할 때만 원래 최상위 위치로
명시적으로 복사한다.
