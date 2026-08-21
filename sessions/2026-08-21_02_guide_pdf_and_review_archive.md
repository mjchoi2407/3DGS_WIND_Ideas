# 2026-08-21 02 개념 가이드 PDF와 리뷰 자료 정리

## Context

현재 Minimal V0 개념 해설을 읽기 쉬운 PDF로 제공하고, 여러 revision 동안 누적된 리뷰 보고서와
LaTeX 중간산출물을 `ideas/` 최상위에서 분리했다. 사용자 파일은 삭제하지 않고 substantive review와
로컬 재생성 자료를 서로 다른 복구 경계에 보존했다.

## Decisions

- `ideas/` 최상위에는 저장소 관리 파일, current sketch의 TeX/Bib/PDF/bundle, current checklist의
  TeX/PDF/bundle, concept guide Markdown/PDF만 둔다.
- 리뷰 본문 17개는 `backup/review_materials_through_2026-08-21/`에 원래 파일명으로 동결한다.
  이 archive는 review provenance이며 canonical method authority가 아니다.
- LaTeX 중간산출물 32개와 `Zone.Identifier` 17개는
  `backup/local_workspace_cleanup_2026-08-21/`에 로컬 전용으로 분리한다. 개념 가이드 PDF의
  파생 XeLaTeX wrapper도 재생성을 위해 이 경계에 함께 보존한다.
- 다운로드 메타데이터에는 URL이 들어갈 수 있으므로 substantive review archive나 Git allowlist에
  넣지 않는다.
- 과거 full-design PDF/ZIP의 `.gitignore` 예외는 파일명 전역 패턴이 아니라 기존 frozen archive의
  정확한 경로로 좁힌다.

## Changed Files

- `wind3dgs_minimal_v0_concept_guide_2026-08-21.pdf`
  - Markdown 내용을 A4 15쪽의 제목·목차·장문 표·흐름도 레이아웃으로 렌더링했다.
- `README.md`
  - concept guide PDF와 두 새 archive 경계를 연결하고 top-level 정리 정책을 명시했다.
- `.gitignore`
  - concept guide PDF와 review provenance PDF의 exact-scope allowlist를 추가하고 predecessor 예외를
    실제 archive 경로로 좁혔다.
- `backup/review_materials_through_2026-08-21/`
  - PDF 리뷰 12개, review source 5개, README와 `SHA256SUMS`를 보존한다.
- `backup/local_workspace_cleanup_2026-08-21/`
  - build artifact 32개, download sidecar 17개, guide wrapper 1개와 local checksum manifest를 보존한다.

원래 `review/` 디렉터리는 10개 파일을 모두 대응 archive로 옮긴 뒤 비어 있음을 확인하고 빈
디렉터리만 제거했다. 연구 내용이 있는 파일은 삭제하지 않았다.

Canonical sketch/checklist TeX, Bib, PDF와 bundle 내용은 이번 정리에서 수정하지 않았다.

## Verification

- Guide PDF build:
  - `latexmk -g -xelatex -shell-escape -interaction=nonstopmode -halt-on-error`
  - 15 pages, A4, 133,487 bytes
  - LaTeX/package fatal error 0, undefined reference/citation 0, rerun warning 0, overfull 0
  - 장문 표의 비치명 underfull 3건만 남으며 시각 검사에서 clipping이나 overlap이 없었다.
- `pdftotext`로 43,261 bytes의 텍스트와 한글을 추출했고 `??` placeholder는 0건이었다.
- `pdftoppm`으로 15쪽 전체를 decode하고 표지, 목차, 전체 흐름·용어 표, 힘 설명, Gate/scope와
  최종 검토 질문 페이지를 raster inspection했다.
- Review archive의 substantive 17개와 local archive의 50개 파일은 이동 전후 SHA-256 일치를
  확인했다. 각 `SHA256SUMS`는 `sha256sum -c`를 통과했다.
- Sketch bundle은 정확히 TeX/Bib/PDF 3개, checklist bundle은 정확히 TeX/PDF 2개이며,
  각 ZIP entry의 SHA-256이 current working file과 일치했다.
- 최종 `ideas/` top-level에는 의도한 current 산출물과 `.git`, `backup`, `sessions`, 정책 파일만
  남고 review PDF, `review/`와 LaTeX 중간산출물은 남지 않았다.

Final SHA-256:

- concept guide Markdown: `18fff70f1492c6ed40f6545ce54654dafa990a2ffd16432455d69a7716e4b804`
- concept guide PDF: `99a9709ebd3d24701b91b08713510aeb605fc4277b9145ba230354e9912a42b8`
- review `SHA256SUMS`: `0d3e90f92efbad4749a5b686586603f7ef037fb48e7510e70a3be36fcb3be61b`
- local cleanup `SHA256SUMS`: `fcf97c69a3a1a455e29bc54bb0a3458e50fba7b881e4625ab9cff0bb8c036083`
- current sketch TeX/PDF/bundle: `e54f6610...` / `3a100e07...` / `fd2d2c2d...`
- current checklist TeX/PDF/bundle: `07e702f0...` / `64782af4...` / `009f00ae...`

## Next

- 현재 방법을 이해하거나 검토할 때는 concept guide를 먼저 읽고, 판단이 필요한 항목은 canonical
  sketch와 checklist stable contract로 돌아간다.
- 과거 리뷰를 다시 확인해야 할 때만 review archive를 사용하며, 과거 지적을 current requirement로
  자동 승계하지 않는다.

이번 작업은 `ideas/` 문서 생성과 정리만 수행했다. Code/experiment 구현, 실행 결과, commit 또는
push는 수행하지 않았다.
