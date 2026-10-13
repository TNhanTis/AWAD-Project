# Đề xuất Dự án: Hệ thống Quản lý Chuỗi Cung ứng Nhà hàng Tự động (Smart Supply)

**Nhóm:** Nguyễn Trọng Nhân, Hồ Hồ Gia Bảo

**Repository:** https://github.com/TNhanTis/AWAD-Project.git

## 1. Problem and users

**Người dùng:** Chị Hoa, quản lý kho của chuỗi nhà hàng ABC (5 chi nhánh).

**Vấn đề:** Mỗi sáng 6h, chị Hoa đi kiểm tồn kho từng kệ, ghi tay vào sổ, rồi gọi điện lần lượt cho 6–8 nhà cung cấp để đặt hàng — mất khoảng 2 tiếng mỗi ngày, và trung bình quên đặt ít nhất 1 nguyên liệu mỗi tuần vì phải nhớ nhẩm số lượng tồn trong lúc gọi điện liên tục.

## 2. The LLM feature and the cost of being wrong

**Tính năng LLM:** Một Agent AI truy hồi dữ liệu bán hàng và tồn kho (RAG), dùng Tool Calling để tính toán định lượng cần đặt, rồi tự động tạo **dự thảo** Đơn đặt hàng (Purchase Order) gửi đến từng nhà cung cấp tương ứng.

**Hậu quả khi mô hình sai:**
- **Ai bị ảnh hưởng:** chủ nhà hàng (tổn thất tài chính trực tiếp) và chị Hoa (người phải giải trình với chủ khi đơn hàng bất thường).
- **Mức độ:** nếu AI ảo giác và đặt dư 100kg thực phẩm tươi sống, nguyên liệu hỏng trong 1–2 ngày gây lãng phí vốn ước tính vài triệu đồng/lần; nếu đặt thiếu, chi nhánh hết nguyên liệu phục vụ, mất doanh thu ngày hôm đó và ảnh hưởng uy tín với khách.
- **Có hoàn tác được không:** không — một khi đơn đã gửi đi và nhà cung cấp xác nhận giao, không thể huỷ.
- **Làm sao biết được khi nó sai:** so sánh số lượng AI đề xuất với mức tiêu thụ trung bình 7 ngày gần nhất của từng nguyên liệu; chênh lệch > 30% được gắn cờ cảnh báo ngay trên màn hình duyệt đơn, trước khi chị Hoa bấm Approve.
- **Guardrail:** AI chỉ tạo **dự thảo**. Chị Hoa bắt buộc xem và bấm Approve thủ công — đây là human gate duy nhất không có ngoại lệ, đặt đúng ở điểm không thể đảo ngược (gửi đơn thật).

## 3. Scope: in and out

**Trong phạm vi (học kỳ này):**
- Thiết kế API contract và schema CSDL cho quản lý nguyên liệu, tồn kho, nhà cung cấp.
- Agent LLM tạo dự thảo đơn hàng (RAG + Tool Calling), kèm guardrail cảnh báo chênh lệch và bộ eval đo độ chính xác đề xuất.
- Harness tự động (lint, test, CI chặn merge), đóng gói Docker, deploy có rollback.
- RBAC phân quyền Quản lý kho (duyệt đơn) vs. Nhân viên (chỉ nhập tồn kho, không duyệt).

**Chủ động bỏ ngoài phạm vi (dù hấp dẫn, nhóm quyết định không làm để tập trung làm sâu tính năng lõi):**
- **Dự báo nhu cầu theo mùa/sự kiện** (ví dụ: tăng đặt hàng dịp lễ) — cần dữ liệu lịch sử dài hơn một học kỳ có thể thu thập, để lại cho phiên bản sau.
- **So sánh giá cùng mặt hàng giữa nhiều nhà cung cấp để tự chọn nguồn rẻ nhất** — là bài toán tối ưu riêng, tách biệt với bài toán "đặt bao nhiêu" mà nhóm đang giải.
- Cổng thanh toán điện tử, app di động cho tài xế giao hàng, hệ thống đặt bàn cho thực khách.

Hai danh sách nhất quán với kế hoạch ở mục 4: mọi mốc đều phục vụ tính năng tạo-và-duyệt đơn hàng, không mốc nào dành cho các phần đã loại bỏ.

## 4. Plan and ownership

| Mốc | Nội dung chính | Người phụ trách | Hạn chót |
|---|---|---|---|
| Checkpoint 1 — Đề xuất & Đặc tả | User story, đặc tả tính năng lõi (tạo/duyệt đơn) | Trọng Nhân | 10/10/2026 |
| Checkpoint 2 — Harness & Cổng chất lượng | File quy ước, lint, test, CI chặn merge | Gia Bảo | 24/10/2026 |
| Checkpoint 3 — Lược đồ dữ liệu & API | Backend, schema migration không downtime (expand/contract) | Trọng Nhân & Gia Bảo | 07/11/2026 |
| Checkpoint 4 — Prototype LLM | Gọi LLM từ backend: streaming, RAG, Tool calling tạo đơn | Trọng Nhân | 21/11/2026 |
| Checkpoint 5 — Eval, Guardrail & Red-team | Bộ eval, human gate, rà soát bảo mật | Gia Bảo | 05/12/2026 |
| Checkpoint 6 — Hoàn thiện, Vận hành & Vấn đáp | Canary/rollback, quan trắc, chuẩn bị bảo vệ | Trọng Nhân & Gia Bảo | 19/12/2026 |

## 5. Risks

**Rủi ro 1 — Rò rỉ ngân sách do Agent lặp vô hạn khi gọi Tool Calling.**
Giảm thiểu, bắt đầu ngay tuần này: áp dụng quyền tối thiểu (least privilege) cho từng tool; đặt hard limit số vòng lặp tối đa của agent (`max_steps = 3`); thiết lập cảnh báo ngân sách trên tài khoản nhà cung cấp model.

**Rủi ro 2 — Lỗi migration làm sập CSDL hệ thống đang chạy.**
Giảm thiểu, bắt đầu ngay khi phác thảo lược đồ dữ liệu: áp dụng chiến lược expand/contract khi đổi schema; thiết lập quy trình rollback trên Docker/CI để khôi phục phiên bản trước trong vòng 5 phút.

## 6. Technology choices

- **Backend (NestJS & PostgreSQL):** cấu trúc chặt chẽ, hỗ trợ transaction và migration CSDL mạnh — phù hợp dữ liệu tồn kho/đơn hàng cần tính nhất quán cao.
- **Frontend (React):** dashboard duyệt đơn cần cập nhật trạng thái gần thời gian thực (đơn mới, cảnh báo chênh lệch) mà không reload trang — hợp với state management của React.
- **LLM Provider (OpenAI `gpt-4o-mini`):** hỗ trợ RAG và Tool Calling tốt, chi phí ~$0.15/1M input token — ước tính dưới 50,000đ/tháng cho toàn bộ lưu lượng thử nghiệm của nhóm trong học kỳ.
- **DevOps & Harness (Docker, GitHub Actions):** CI/CD miễn phí cho repo nhóm nhỏ, đủ để làm cổng chất lượng chặn merge và canary deploy.
- **Hosting (Railway/Render free tier thay vì AWS):** giữ chi phí bằng 0 trong học kỳ, tránh rủi ro phát sinh phí ngoài dự kiến như khi dùng AWS; sẽ cân nhắc AWS nếu cần mở rộng sau khi môn kết thúc.