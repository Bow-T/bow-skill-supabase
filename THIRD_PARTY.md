# Nguồn bên ngoài

Skill chép từ repo khác được ghi ở đây: lấy từ đâu, tại commit nào, giấy phép gì, và đã sửa gì.
Bản chép là **ảnh chụp tại một commit** — bow-agent không tự kéo bản mới của nguồn. Muốn cập nhật
thì làm lại các bước ở cuối file, đọc lại nội dung, rồi mới gắn tag.

| Skill | Nguồn | Commit | Giấy phép | Đã sửa |
| --- | --- | --- | --- | --- |
| `supabase` | [supabase/agent-skills](https://github.com/supabase/agent-skills) `skills/supabase` (v0.1.2) | `544bfc56c89afe2b87b20017a59b2c6e9502a1fb` (2026-09-28) | MIT | Bỏ mục "Reference Guides" cùng `references/skill-feedback.md`, `assets/feedback-issue-template.md`, `CHANGELOG.md` |
| `supabase-postgres-best-practices` | [supabase/agent-skills](https://github.com/supabase/agent-skills) `skills/supabase-postgres-best-practices` (v1.1.1) | `544bfc56c89afe2b87b20017a59b2c6e9502a1fb` (2026-09-28) | MIT | Bỏ `CHANGELOG.md`, `references/_contributing.md`, `references/_template.md`; nội dung còn lại nguyên văn |

Văn bản giấy phép: [`LICENSES/supabase-agent-skills-MIT.txt`](LICENSES/supabase-agent-skills-MIT.txt).

## Vì sao bỏ những phần đó

- **Mục "Reference Guides" của `supabase`** bảo agent soạn và **mở issue lên repo `supabase/agent-skills`**
  khi người dùng chê skill. Đó là hành động gửi nội dung phiên làm việc ra một repo công khai — không
  hợp với một agent đang cầm quyền trên repo của dự án, nên bỏ cả mục lẫn hai file nó trỏ tới.
- Các file `_contributing.md`, `_template.md`, `CHANGELOG.md` chỉ dành cho người soạn skill ở nguồn;
  agent không cần, để lại chỉ tốn chỗ.

## Đã soát gì trước khi chép (2026-10-02)

- Không có script / file thực thi nào trong hai skill — chỉ Markdown.
- Không có ký tự vô hình (tag Unicode, zero-width, bidi).
- Đường link chỉ trỏ tới `supabase.com`, `postgresql.org`, `wiki.postgresql.org`, `github.com/supabase`.
- Lệnh mạng duy nhất skill bảo chạy: đọc tài liệu ở `supabase.com` và một lệnh `curl` kiểm
  `mcp.supabase.com` còn sống.

## Cập nhật từ nguồn

```bash
git clone --depth 1 https://github.com/supabase/agent-skills /tmp/up && cd /tmp/up && git rev-parse HEAD
# so từng skill với bản ở đây, ĐỌC phần khác biệt (đặc biệt lệnh / link / mục mới), rồi chép đè
diff -r /tmp/up/skills/supabase-postgres-best-practices skills/supabase-postgres-best-practices
diff <(sed '/^## Reference Guides/,$d' /tmp/up/skills/supabase/SKILL.md) skills/supabase/SKILL.md
```
Sửa bảng ở trên (commit + ngày), tăng `version` trong `bow-skill.json`, gắn tag mới.
