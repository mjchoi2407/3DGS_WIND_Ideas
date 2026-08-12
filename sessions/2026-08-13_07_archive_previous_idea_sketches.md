# 2026-08-13 07 이전 아이디어 스케치 백업 이동

## 요청

오늘 작성한 `Topology-Distilled, Error-Triggered Global--Local Wind Dynamics` 문서만 `ideas/`의 현재 아이디어 스케치로 남기고, 이전 아이디어 스케치를 삭제하지 않은 채 별도 백업 폴더로 이동한다.

## 백업 위치

`backup/previous_idea_sketches_before_2026-08-13/`

## 이동 범위

기존 파일 45개를 원래 이름 그대로 이동했다.

- `idea_sketch.*`
- `3dgs_wind_response_pipeline_2026-05-12.*`
- `anchor_method_revision_notes_*`
- `implementation_checklist.*`
- 2026-08-05 아이디어 bundle과 다운로드 metadata
- 2026-08-12 adaptive RSH residual bundle과 다운로드 metadata
- 기존 문서용 `refs.bib`, `changelog.md`
- 위 문서의 LaTeX build artifact

기존 `idea_sketch.tex`, `idea_sketch.pdf`, `changelog.md`에 있던 수정 상태도 내용 변경 없이 백업 위치에 보존했다. 삭제한 파일은 없다.

## 유지한 범위

- 현재 `.tex`, 전용 `.bib`, `.pdf`, bundle과 현재 LaTeX build artifact
- `README.md`, `AGENTS.md`, `.gitignore`
- `sessions/`

`README.md`의 현재 작업본 안내를 새 문서명과 백업 위치에 맞게 갱신했다.

## 검증

- 백업 폴더: 기존 파일 45개와 백업 README 1개, 총 46개
- `ideas/` 최상위에 이전 스케치 계열 파일이 남지 않았음을 확인
- 현재 문서를 `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 강제 재빌드하여 예전 `refs.bib` 의존성이 없음을 확인
- 재빌드된 PDF로 현재 전달 bundle을 갱신
- bundle 내부 세 파일의 압축 무결성을 다시 검사
