# Roadmap — Template

Copy to `docs/superpowers/plans/YYYY-MM-DD-<topic>-roadmap.md`. Sections marked
**[REQUIRED]** always appear; the rest are added when the project needs them.

Prose stays in the project's working language.

---

```markdown
# Lộ trình <tên công việc>

> **Đây là plan lộ trình, không phải implementation plan.** Plan chi tiết của mỗi
> phase viết riêng khi bắt đầu phase đó. Không viết trước — hiểu biết sẽ thay đổi
> sau mỗi phase.

**Ngày lập:** YYYY-MM-DD  ·  **Căn cứ:** <tài liệu yêu cầu>
**Hồ sơ hệ thống:** docs/superpowers/system-profile.md
**Mục tiêu:** <1–2 câu, kết quả cuối cùng>

## Phạm vi                                            [REQUIRED]
Cái gì được sửa, cái gì CẤM đụng vào. Nếu một phase phát hiện cần đổi hợp đồng
với phần cấm đụng thì dừng lại và báo — không tự sửa.

## Định hướng đã chốt                                  [REQUIRED]
Các quyết định xuyên suốt mọi phase, mỗi cái 1–2 dòng kèm lý do.

## Bản đồ phase                                        [REQUIRED]

P0 <nền móng>
   └─> P1 <...>
        └─> P2 <...>
             ├─> P3 <...>
             └─> P4 <...>        ← MỐC DÙNG ĐƯỢC

**Mốc dùng được:** hết P<n> — <người dùng làm được gì tại mốc này>.

## Chi tiết từng phase                                 [REQUIRED]

### P<n> — <tên>
- **Mục tiêu:** <1 câu>
- **Phạm vi:** <những gì nằm trong phase này>
- **Tái sử dụng:** <copy/tham khảo từ đâu, hoặc "viết mới">
- **Đã chốt:** <quyết định đã ra, để phiên sau không lật lại>
- **Định nghĩa hoàn thành:** <quan sát được — không phải "code chạy">

## Những gì loại bỏ hẳn                                [nếu có]
Bảng: bỏ cái gì | lý do. Ngăn phase sau vô tình làm lại.

## Bẫy kỹ thuật bắt buộc mang theo                     [nếu có]
Bảng: bẫy | hệ quả nếu quên | thuộc phase nào.

## Rủi ro                                              [nếu có]
Bảng: rủi ro | mức | cách xử lý.

## Cách làm việc theo phase                            [nếu có]
Ai viết plan, ai thực thi, ai kiểm — chỉ ghi khi khác mặc định của Conductor.
```

---

## A worked example

From a report-service rebuild — nine phases, foundation upward, with the usable
milestone deliberately placed before the end:

```text
P0 Khung service
   └─> P1 Định danh, phân quyền, lưu vết, file
        └─> P2 Nạp dữ liệu warehouse
             └─> P3 Danh mục chỉ tiêu + engine tính
                  └─> P3b Hiệu chỉnh theo requirement mới   ← PHÁT SINH sau P3
                       ├─> P4 Rollup ngày
                       └─> P5 Mẫu báo cáo + 4 loại tag
                       └─> P6 Vòng đời báo cáo + snapshot
                            └─> P7 Xuất file + ký số        ← MỐC DÙNG ĐƯỢC
                                 ├─> P8 Lịch + nhắc nhở
                                 └─> P9 Cổng ngoài + chuyển đổi
```

Read what that structure gets right:

- **Every phase unlocks the next.** Without P2 there is no data for P3's engine
  to compute; without P3's catalogue there is nothing for P5's templates to bind.
- **The usable milestone sits at P7, not P9.** By then users create reports,
  figures aggregate, files export signed. P8/P9 are automation and integration.
- **P3b appeared mid-flight** and was inserted at its dependency position, with
  its origin date recorded — not stuffed into the running phase.
- **P3 alone ran 11 tasks across 6 executor rounds.** Phases are not
  session-sized; that phase spanned many sessions and stayed one layer.

A phase entry from the same document, showing the level of detail that belongs in
a roadmap (and no more):

> ### P0 — Khung service
>
> **Mục tiêu:** repo mới chạy được, có health check, deploy được lên k3s dev.
>
> **Phạm vi:** khởi tạo repo; cấu hình TypeScript/ESLint; kết nối Mongo; logger;
> health endpoint; `src/common` (enum tập trung, mã lỗi, i18n); Dockerfile; Helm
> chart; script CI.
>
> **Copy từ hệ cũ:** `src/common`, `src/config`, `src/mongo`, `src/loggers`,
> `helm/`, `Dockerfile` — sửa tên service, cắt enum không còn dùng.
>
> **Định nghĩa hoàn thành:** pod chạy trên k3s dev, `/health` trả 200, CI xanh.

Note what is absent: no file list, no interface sketches, no task breakdown. Those
belong to P0's detailed plan, written when P0 starts.
