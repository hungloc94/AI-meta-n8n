# CASE-001: Polling thay vì Webhook

## Vấn đề
Cần cơ chế nhận message Telegram cho bot reporting.

## Nguyên nhân
Webhook cần public exposure (Cloudflare tunnel hoặc domain), khó test local trong giai đoạn recovery. Rủi ro bảo mật khi expose endpoint ra internet.

## Cách xử lý
Dùng Polling (getUpdates) — bot chủ động hỏi Telegram server theo interval.

## Bài học
- Polling yêu cầu staticData/offset persistence rất cẩn thận để tránh duplicate processing.
- Nếu offset bị reset → toàn bộ message cũ sẽ bị replay.
- Trade-off chấp nhận được cho giai đoạn MSP recovery/testing.

## Status
Accepted.
