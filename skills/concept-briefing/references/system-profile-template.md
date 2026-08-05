# System Profile — Template

Copy to `docs/superpowers/system-profile.md`. Prose stays in the project's working
language; the headings below are Vietnamese because that is this project's.

Every line carries exactly one marker: `~` inferred by Claude from the repo,
`✓` confirmed by the user. The markers exist so a machine guess never freezes
into "fact" and then propagates into every downstream prompt.

The four **bold** lines below must be answered by the user directly. Draft the
rest and have them confirmed in one batch.

---

```markdown
# System Profile: <tên hệ thống>

**Cập nhật:** YYYY-MM-DD
**Trạng thái:** CHƯA CHỐT (nháp Claude suy từ repo) | ĐÃ CHỐT — user xác nhận YYYY-MM-DD
**Ký hiệu:** `~` = Claude suy từ repo, chưa ai xác nhận · `✓` = user đã chốt

## Quy mô & tải
- **Người dùng:** <số lượng, nội bộ hay công khai>
- Tải: <req/s hoặc job/ngày, dung lượng dữ liệu>
- Tăng trưởng 12 tháng tới: <ổn định | tăng dần | nhân nhiều lần>
- **Scale:** <1 instance đủ | cần scale ngang | đã scale ngang>

## Ưu tiên khi tradeoff
**Xếp hạng, dùng khi hai yêu cầu xung đột (1 = cao nhất):**
1. <vd: đúng đắn số liệu>
2. <vd: tốc độ ra tính năng>
3. <vd: hiệu năng>
4. <vd: chi phí hạ tầng>

**Không đánh đổi:** <ranh giới cứng, vd: không được sai số liệu kỳ đã chốt>

## Định hướng code
- Mức trừu tượng: <trực tiếp ít lớp | có lớp mở rộng sẵn>
- Mức test kỳ vọng: <smoke | unit cho logic nghiệp vụ | phủ cao>
- Bị coi là over-engineering ở đây: <mô tả cụ thể>

## Ràng buộc vận hành
- Môi trường deploy: <...>
- Uptime / cửa sổ downtime cho phép: <...>
- Ai vận hành, mức giám sát: <...>

## Dữ liệu & tuân thủ
- Dữ liệu nhạy cảm: <...>
- Audit / backup / lưu vết: <...>

## Biên hệ thống
- Hợp đồng KHÔNG được phá: <API công khai, schema, job, consumer ngoài>
- Phụ thuộc ngoài: <...>
```

---

## What to draft from the repo

| Section | Look at |
|---------|---------|
| Quy mô & tải | Data volumes in migrations/seeds, replica counts in k8s/compose, rate limits in config |
| Định hướng code | Existing abstraction depth, test directory size and style, lint config |
| Ràng buộc vận hành | CI/CD config, Helm/k8s manifests, healthcheck and probe setup |
| Dữ liệu & tuân thủ | Field names suggesting PII, audit tables, encryption helpers, retention jobs |
| Biên hệ thống | Public route files, published schemas, message topics, cron/job entry points |

Nothing in the repo tells you the real user count, the growth expectation, or
which priority wins a conflict. Those are the questions worth the user's time.

## What a good one reads like

Concrete numbers, no hedging — the shape to aim for:

> Service **rất ít người dùng, quy mô nhỏ, gần như không bao giờ scale**. Mọi
> quyết định thiết kế phải chọn phương án đơn giản hơn khi hai phương án cùng
> đáp ứng yêu cầu.
>
> | Hạng mục | Số đo được |
> |---|---|
> | Dữ liệu | <số document / bảng, đo ngày nào> |
> | Người dùng hệ thống | <số thật, nội bộ hay công khai> |
> | Số bản chạy service | <n replica> |
> | Ca dùng nặng nhất | <mô tả — thứ phải thiết kế để chịu được> |

**Measured numbers beat adjectives.** "Nhỏ" is arguable and gets re-litigated
every phase; "1 replica, 200 internal users, heaviest case is the annual report"
is not, and it settles a dozen architecture arguments before they start.

Write the row for "heaviest realistic case" even when it looks obvious. It is the
one number that decides whether a design is adequate, and the one most often left
as a feeling.
