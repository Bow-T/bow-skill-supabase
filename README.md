# bow-skill-supabase

Bộ skill cho công nghệ **Supabase**, dùng với [bow-agent](https://github.com/Bow-T/bow-agent).
Đây là repo mẫu của hướng **mỗi công nghệ một repo**: bộ này không gắn với framework nào, một dự án
tick nó CÙNG với bộ framework của mình (Flutter, Next.js, React Native…) ở ô **Công nghệ**.

## Cấu trúc

```
bow-skill-supabase/
├── bow-skill.json        # manifest: id "supabase" + quy tắc tự nhận diện repo
├── THIRD_PARTY.md        # skill nào chép từ đâu, commit nào, giấy phép gì, đã sửa gì
├── LICENSES/             # văn bản giấy phép của nguồn bên ngoài
└── skills/
    ├── supabase/                          # ⬇ nguồn chính thức — mọi việc đụng tới Supabase
    ├── supabase-postgres-best-practices/  # ⬇ nguồn chính thức — 31 quy tắc Postgres (index, RLS, khoá…)
    ├── supabase-migration/                # team — VIẾT thay đổi schema đúng cách
    └── supabase-security-review/          # team — SOÁT thay đổi backend trước khi commit
```

| Skill | Dùng khi | Nguồn |
| --- | --- | --- |
| `supabase` | bất kỳ việc gì có Supabase: Auth, Edge Function, Storage, CLI, MCP, gỡ lỗi | Supabase (MIT) |
| `supabase-postgres-best-practices` | viết / sửa thứ gì nằm trong Postgres; truy vấn chậm | Supabase (MIT) |
| `supabase-migration` | tạo / sửa bảng, cột, policy RLS, trigger | team Bow |
| `supabase-security-review` | trước khi commit thay đổi dưới `supabase/` | team Bow |

Hai skill của team tách ra từ `bow-skill-flutter`, đã gỡ phần riêng của một sản phẩm (mã ticket,
tên hàm, tên migration cụ thể). Cổng `scripts/check-quest.sh` chỉ chạy **khi repo có script đó**;
repo không có thì dùng `supabase db advisors`. Ví dụ policy theo khuyến nghị của nguồn chính thức
(`to authenticated` + `(select auth.uid())`).

**Khi hai nguồn nói khác nhau, quy ước team thắng.** Skill `supabase` cho phép thử schema bằng
`execute_sql` rồi mới sinh migration; `supabase-migration` ghi rõ: chỉ được làm vậy trên stack LOCAL,
còn với project remote thì file migration đã commit phải có trước.

## Tự nhận diện

bow-agent tự tick bộ này cho dự án chưa khai công nghệ khi repo khớp một trong các quy tắc ở
`targets.detect`:

| Quy tắc | Nghĩa |
| --- | --- |
| có `supabase/config.toml` | repo chứa project Supabase (tìm cả trong thư mục con) |
| `pubspec.yaml` chứa `supabase_flutter` | app Flutter gọi Supabase |
| `package.json` chứa `"@supabase/supabase-js"` hoặc `"@supabase/ssr"` | web / React Native gọi Supabase |

Chỉ soi file git đã track. App Flutter gọi REST API riêng (không có gói Supabase) sẽ KHÔNG khớp.

## Đăng ký vào bow-agent

Thêm vào `stacks` của `~/.bow-agent/registry.json` (bow chỉ tải repo có trong danh sách này, đúng tag đã ghim):

```json
{ "id": "supabase", "label": "Supabase", "repo": "github.com/Bow-T/bow-skill-supabase", "ref": "v1.0.0" }
```

Lưu ý trong lúc chuyển tiếp: bundle ghép cũ "Flutter + Supabase" cũng mang `supabase-migration` và
`supabase-security-review`. Tick cả hai bộ thì bản của mục đứng TRƯỚC trong danh sách được dùng —
đặt mục `supabase` lên trước nếu muốn dùng bản ở đây.

Thử local trước khi đẩy GitHub: `"repo": "file:///đường/dẫn/tới/bow-skill-supabase"`.

## Phát hành phiên bản mới

Sửa skill qua PR → tăng `version` trong `bow-skill.json` → gắn tag (`git tag v1.0.1 && git push --tags`)
→ cập nhật `ref` trong registry. bow tải theo tag và cache theo `<id>@<ref>`, nên sửa nội dung mà giữ
nguyên tag sẽ KHÔNG được tải lại. Cập nhật skill từ nguồn bên ngoài: xem cuối `THIRD_PARTY.md`.
