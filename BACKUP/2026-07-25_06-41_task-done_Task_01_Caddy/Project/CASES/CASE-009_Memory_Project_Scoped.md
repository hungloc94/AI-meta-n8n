# CASE-009: Project Memory phải Project-Scoped

## Vấn đề
Context bị phân tán nếu dùng global memory folder chung cho nhiều project.

## Nguyên nhân
Global `AI_MEMORY` folder chứa nhiều project khác nhau. AI agent dễ nhầm lẫn context, mất authority hierarchy, và khó handoff.

## Cách xử lý
Memory self-contained trong project folder: `D:\AI_Project\Obdisian_2\My Vault\Project\AI_Meta_n8n_autoamation`. Không dùng global `AI_MEMORY`.

## Bài học
- Mọi AI agent phải dùng đúng folder này.
- Giữ context, authority, và handoff clarity.
- File trong project folder là nguồn sự thật cho project này.

## Status
Accepted.
