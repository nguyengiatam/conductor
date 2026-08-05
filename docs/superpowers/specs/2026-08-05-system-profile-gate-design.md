# System Profile Gate (concept-briefing v2) — Design

**Goal:** Sửa lại phase lấy concept của Conductor. Hiện `concept-briefing` chỉ đo
**quy mô của yêu cầu** (task này to hay nhỏ). Thiếu hẳn thứ quan trọng hơn:
**hồ sơ của hệ thống** — hệ thống lớn hay nhỏ, bao nhiêu người dùng, có cần scale
không, khi hai yêu cầu xung đột thì ưu tiên cái nào. Đây mới là thứ quyết định
lựa chọn phương án code, và nó phải được **chốt với người dùng**, không phải suy
đoán từ repo.

## Problem

`concept-briefing` hiện tại (v0.2.0) sinh ra `concept-brief.md` với ba dòng: quy
mô ước lượng, mức độ nghiệp vụ, kỳ vọng người yêu cầu. Cả ba đều nói về **request**,
không nói gì về **hệ thống đang được sửa**. Hệ quả:

- Khi `brainstorming` chọn giữa hai phương án (ví dụ: cache trong process vs
  Redis; xử lý đồng bộ vs queue), nó không có căn cứ nào để quyết — không biết
  hệ thống phục vụ 200 người nội bộ hay 50k người ngoài, không biết có được phép
  làm mỏng hay không.
- Executor ngoài (Codex/agy) hoàn toàn mù bối cảnh này. Prompt handoff hiện chỉ
  trích dòng calibration quy mô request, nên executor mặc định over-engineer hoặc
  under-engineer theo thói quen của nó.
- `adversarial-review-to-go` không phân biệt được rủi ro thật với rủi ro giả —
  bắt lỗi scale trên một hệ thống nội bộ 200 user là nhiễu, bỏ qua nó trên hệ
  thống công khai là thảm hoạ.
- Cùng một thông tin bối cảnh bị hỏi lại (hoặc tệ hơn: bị đoán lại, khác đi) ở
  mỗi request mới, vì không có chỗ nào lưu.

## Scope

**In scope:**
1. Một artifact mới, sống lâu, một-lần-mỗi-project:
   `docs/superpowers/system-profile.md`.
2. Cơ chế lập hồ sơ: Claude suy nháp từ repo → **người dùng chốt** → mới được đi
   tiếp. Đây là **gate cứng**.
3. Sửa `concept-briefing` thành skill hai bước (bước 0 đảm bảo hồ sơ, bước 1 giữ
   calibration quy mô request như cũ nhưng rút gọn).
4. Cập nhật các skill hạ nguồn để đọc hồ sơ: `orchestrating-executors`,
   `adversarial-review-to-go`; cập nhật `using-conductor` và `README`.

**Out of scope:**
- Không đụng vào `checkpoint-verification`, `convention-commit-gate`.
- Không tách thành skill riêng — hai việc nằm chung trong `concept-briefing`
  (xem "Đóng gói" bên dưới).
- Hồ sơ không chứa kiến trúc, vocabulary, hay task dependency — những thứ đó vẫn
  thuộc spec/plan do `brainstorming`/`writing-plans` sinh ra.

## Hai artifact, hai vòng đời

| | `system-profile.md` | `concept-brief.md` |
|---|---|---|
| Phạm vi | Cả project | Một request |
| Vòng đời | Nhiều tháng, sửa khi hệ thống đổi | Một lần dùng rồi thôi |
| Trả lời câu hỏi | "Hệ thống này là loại gì, ưu tiên gì?" | "Việc này to tới đâu?" |
| Nguồn sự thật | **Người dùng chốt** | Claude tự ước lượng |
| Vị trí | `docs/superpowers/system-profile.md` (không gắn ngày) | `docs/superpowers/plans/YYYY-MM-DD-<topic>-concept-brief.md` |

Lý do tách: hai thứ có vòng đời khác hẳn nhau. Gộp chung sẽ khiến bối cảnh hệ
thống bị chép lại ở mỗi request và trôi lệch dần giữa các bản.

## Nội dung `system-profile.md`

```markdown
# System Profile: <tên hệ thống>

**Cập nhật:** YYYY-MM-DD
**Trạng thái:** CHƯA CHỐT (nháp Claude suy từ repo) | ĐÃ CHỐT — user xác nhận YYYY-MM-DD
**Ký hiệu:** `~` = Claude suy từ repo, chưa ai xác nhận · `✓` = user đã chốt

## Quy mô & tải
- Người dùng: <số lượng, nội bộ hay công khai>
- Tải: <req/s hoặc job/ngày, dung lượng dữ liệu>
- Tăng trưởng 12 tháng tới: <ổn định | tăng dần | nhân nhiều lần>
- Scale: <1 instance đủ | cần scale ngang | đã scale ngang>

## Ưu tiên khi tradeoff
Xếp hạng, dùng khi hai yêu cầu xung đột (1 = cao nhất):
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

Toàn bộ prose bằng tiếng Việt, khớp ngôn ngữ làm việc của project.

Ký hiệu `~` / `✓` tồn tại để một suy đoán của máy không bao giờ đóng băng thành
"sự thật" rồi lan xuống mọi prompt phía sau. Mỗi dòng phải mang đúng một trong
hai ký hiệu.

## Gate cứng: hồ sơ phải được người dùng chốt

Đây là điểm trung tâm của thiết kế, không phải chi tiết phụ.

- `concept-briefing` **không được kết thúc** khi `system-profile.md` còn ở trạng
  thái `CHƯA CHỐT`.
- `superpowers:brainstorming` **không được bắt đầu chọn phương án** khi hồ sơ
  chưa chốt. Khác hẳn fallback mềm mà `orchestrating-executors` đang dùng cho
  `concept-brief.md` — ở đây là chặn cứng, vì lựa chọn phương án code phụ thuộc
  nặng vào hồ sơ.
- **Bốn dòng bắt buộc người dùng trả lời trực tiếp**, Claude tuyệt đối không được
  tự điền rồi coi là xong:
  1. Người dùng thật (số lượng, nội bộ hay công khai)
  2. Nhu cầu scale
  3. Thứ tự ưu tiên khi tradeoff
  4. Cái không được đánh đổi

  Các dòng còn lại Claude điền nháp, người dùng xác nhận theo lô.
- **Im lặng không phải là đồng ý.** Không có đường tắt "user không phản đối nên
  coi như chốt".
- Nếu người dùng chưa quyết được một dòng: ghi `CHƯA CHỐT` ngay tại dòng đó,
  không chốt cả file bằng một dòng treo. Cấm mọi quyết định kiến trúc dựa trên
  dòng đó; khi công việc chạm tới nó thì phải quay lại hỏi.

## Đóng gói: một skill, hai bước

`concept-briefing` giữ nguyên tên và vị trí trong arc. Bên trong chia hai bước.

### Bước 0 — Đảm bảo hồ sơ hệ thống

1. Tìm `docs/superpowers/system-profile.md`.
2. **Đã có và ĐÃ CHỐT** → đọc. Kiểm tra tính tươi bằng **mâu thuẫn quan sát
   được**, không bằng ngưỡng thời gian cứng: repo nói khác hồ sơ (ví dụ hồ sơ ghi
   "1 instance đủ" nhưng repo vừa thêm HPA / thêm worker), hoặc chính request
   đang xử lý ngụ ý khác (ví dụ yêu cầu "chịu 10k user đồng thời" trên hồ sơ ghi
   200 user nội bộ). Có mâu thuẫn → hỏi đúng một dòng xác nhận cho dòng đó, không
   mở lại toàn bộ bảng hỏi. Không mâu thuẫn → dùng luôn, không hỏi gì.
3. **Đã có nhưng CHƯA CHỐT** → tiếp tục vòng chốt đang dở, không đi tiếp.
4. **Chưa có** → Claude quét repo để điền nháp: `README`, `CLAUDE.md`,
   `docker-compose` / manifest k8s, file config, thư mục migration, cấu hình CI,
   manifest package, số lượng service. Mọi dòng suy được đánh `~`. Sau đó trình
   cho người dùng **một lượt gộp** gồm: bốn dòng bắt buộc + những dòng nháp có độ
   tin cậy thấp. Ghi file với trạng thái `ĐÃ CHỐT`, commit.

Nguyên tắc: hỏi theo lô, không hỏi sáu vòng. Chi phí của bước này là một lần cho
cả project, nên chấp nhận được — nhưng vẫn phải gọn.

### Bước 1 — Calibration quy mô request

Giữ cơ chế hiện tại của `concept-brief.md` (1–3 câu hỏi nhẹ, hoặc không hỏi gì
với request hiển nhiên nhỏ; xác nhận bằng một dòng, không phải gate spec-review).
Rút gọn: brief **không lặp lại** bối cảnh hệ thống nữa, chỉ tham chiếu tới
`system-profile.md`. Ba dòng còn lại: quy mô ước lượng, mức độ nghiệp vụ, kỳ vọng
người yêu cầu — cộng phần hàm ý hiệu chỉnh như cũ.

## Tác động xuống hạ nguồn

- **`superpowers:brainstorming`**: đọc cả hai. Hồ sơ quyết định *chọn phương án
  nào* (được phép làm mỏng tới đâu, có cần lớp mở rộng không); brief quyết định
  *đào sâu tới đâu* (bao nhiêu approach, hỏi bao nhiêu câu).
- **`orchestrating-executors`**: prompt handoff trích thẳng từ hồ sơ ba thứ —
  thứ tự ưu tiên tradeoff, biên không được phá, mức test kỳ vọng. Đây là chỗ giá
  trị nhất, vì executor ngoài hoàn toàn mù bối cảnh này. Bullet hiện có về
  `concept-brief.md` giữ nguyên, thêm bullet mới cho hồ sơ.
- **`adversarial-review-to-go`**: reviewer nhận hồ sơ để phân biệt rủi ro thật
  với rủi ro giả, và để biết đâu là ranh giới không được phá.
- **`checkpoint-verification`, `convention-commit-gate`**: không đổi.

## Cập nhật hồ sơ về sau

- Khi phát hiện thực tế lệch hồ sơ (ví dụ hoá ra hệ thống có 50k user, không phải
  200), Claude sửa file và đổi ngày cập nhật, không chờ được yêu cầu.
- Khi một dòng bị sửa, Claude **chủ động rà lại** các quyết định kiến trúc đã dựa
  trên dòng cũ và báo lại cho người dùng — không im lặng đi tiếp. Một hồ sơ đổi
  giữa chừng có thể làm vô hiệu phương án đã chọn.
- Dòng sửa bởi người dùng giữ `✓`; dòng Claude tự sửa từ quan sát mới quay về `~`
  cho tới khi người dùng xác nhận.

## Các file phải sửa

| File | Thay đổi |
|---|---|
| `skills/concept-briefing/SKILL.md` | Viết lại: hai bước, gate cứng, template hồ sơ, quy tắc `~`/`✓`, cập nhật Red Flags |
| `skills/orchestrating-executors/SKILL.md` | Thêm bullet vào Handoff Prompt Checklist: trích ưu tiên tradeoff + biên không được phá + mức test từ hồ sơ |
| `skills/adversarial-review-to-go/SKILL.md` | Reviewer đọc hồ sơ để hiệu chỉnh cái gì đáng coi là finding |
| `skills/using-conductor/SKILL.md` | Cập nhật arc + bảng "When to Use Which Skill" |
| `README.md` | Cập nhật mô tả `concept-briefing` + arc |
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json` | Bump `0.2.0` → `0.3.0` |

## Edge cases

- **Repo nghèo dấu vết** (không README, không compose): Claude điền được rất ít,
  phần lớn dòng để trống và hỏi. Chấp nhận — thà hỏi còn hơn đoán bừa.
- **Conductor áp dụng giữa chừng, chưa từng có hồ sơ**: gate vẫn cứng — lập hồ sơ
  trước khi brainstorming. Đây là lần duy nhất phải trả chi phí này.
- **Request cực nhỏ (sửa typo) trên project chưa có hồ sơ**: vẫn phải lập hồ sơ.
  Đây là đánh đổi có chủ đích của thiết kế — hồ sơ là tài sản của project, không
  phải chi phí của request; lập một lần rồi mọi request sau đều hưởng.
- **Nhiều hệ thống trong một repo (monorepo)**: một `system-profile.md` cho mỗi
  đơn vị triển khai độc lập, đặt cạnh đơn vị đó; nếu cả monorepo triển khai chung
  thì một file ở gốc.
- **Người dùng từ chối trả lời**: không có đường vòng. Ghi `CHƯA CHỐT` tại dòng
  đó và dừng phần công việc phụ thuộc vào nó; các phần khác vẫn chạy được.

## Verification Plan

Thay đổi này là file skill/doc, không phải code ứng dụng, nên verify bằng dry run
thật:

1. Chạy `concept-briefing` trên một project chưa có hồ sơ (market-report) →
   xác nhận Claude sinh được bản nháp từ repo, hỏi đúng bốn dòng bắt buộc theo
   một lượt gộp, và **không đi tiếp** khi chưa được chốt.
2. Chạy lại trên project đã có hồ sơ ĐÃ CHỐT → xác nhận không hỏi lại, chỉ đọc.
3. Kiểm tra một prompt handoff do `orchestrating-executors` sinh ra có thật sự
   chứa thứ tự ưu tiên tradeoff và biên không được phá.
4. Xác nhận bốn skill còn lại không đổi hành vi ngoài các mục đã liệt kê ở bảng
   trên.
