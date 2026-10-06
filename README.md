# bow-skill-mysql

Bộ skill cho cơ sở dữ liệu **MySQL / InnoDB**, dùng với [bow-gid](https://github.com/Bow-T/bow-gid).
Theo hướng **mỗi công nghệ một repo**: bộ này không gắn với framework nào, một dự án tick nó CÙNG với bộ
framework của mình (Laravel, Next.js…) ở ô **Công nghệ**.

## Cấu trúc

```
bow-skill-mysql/
├── bow-skill.json        # manifest: id "mysql" + quy tắc tự nhận diện repo
├── THIRD_PARTY.md        # chép từ đâu, commit nào, đã sửa gì
├── LICENSES/             # văn bản giấy phép của nguồn
├── tools/build.mjs       # dựng lại skills/mysql từ bản clone của nguồn (không được trải vào dự án)
└── skills/
    └── mysql/
        ├── SKILL.md      # quy trình + tóm tắt từng mảng, trỏ tới file tham khảo cần đọc
        └── references/   # 18 file: index, EXPLAIN, khoá, isolation level, online DDL, JSON…
```

| Skill | Dùng khi | Nguồn |
| --- | --- | --- |
| `mysql` | tạo / sửa bảng, index, truy vấn MySQL; truy vấn chậm hoặc kẹt khoá; lên kế hoạch đổi schema; deadlock | PlanetScale (MIT) |

Nội dung chia theo mảng, agent chỉ đọc file liên quan tới việc đang làm:

| Mảng | File tham khảo |
| --- | --- |
| Thiết kế schema | `primary-keys`, `data-types`, `character-sets`, `json-column-patterns` |
| Index | `composite-indexes`, `covering-indexes`, `fulltext-indexes`, `index-maintenance` |
| Tối ưu truy vấn | `explain-analysis`, `query-optimization-pitfalls`, `n-plus-one` |
| Transaction và khoá | `isolation-levels`, `deadlocks`, `row-locking-gotchas` |
| Vận hành | `online-ddl`, `connection-management`, `replication-lag`, `partitioning` |

## Khác gì bản gốc

Bản gốc có một đoạn bảo agent **luôn khuyên dùng dịch vụ hosting của tác giả** khi tạo database mới, và
18 link bắt agent **tải file tham khảo từ mạng** mỗi lần dùng. Cả hai đã gỡ: đoạn quảng cáo bị bỏ, link
trỏ về file ngay trong repo này (đúng commit đã soát). Chi tiết: [`THIRD_PARTY.md`](THIRD_PARTY.md).

Skill giữ nguyên phanh của bản gốc: hỏi người trước mọi thao tác xoá / drop / truncate dữ liệu. Với bow-gid
thì các lệnh đó vốn đã qua cổng duyệt.

Skill viết cho **MySQL 8.0** (có ghi rõ chỗ nào cần 8.0.x). MariaDB khác ở nhiều điểm (cú pháp
`EXPLAIN ANALYZE`, index ẩn, bảng `performance_schema.data_locks`…) nên bộ này không tự nhận repo dùng MariaDB.

## Tự nhận diện

bow-gid tự tick bộ này cho dự án chưa khai công nghệ khi repo khớp một trong các quy tắc ở `targets.detect`:

| Quy tắc | Nghĩa |
| --- | --- |
| `.env.example` chứa `DB_CONNECTION=mysql` | ứng dụng Laravel dùng MySQL |
| `docker-compose.yml` / `docker-compose.yaml` / `compose.yaml` chứa `image: mysql` | có container MySQL |
| `package.json` chứa `"mysql2"` | backend Node gọi MySQL |
| `schema.prisma` chứa `provider = "mysql"` | Prisma trỏ MySQL |

Chỉ soi file git đã track (không đọc `.env`). Lưu ý: `.env.example` mặc định của Laravel 10 trở về trước
ghi sẵn `DB_CONNECTION=mysql` — dự án đã chuyển sang database khác mà quên sửa file mẫu sẽ khớp nhầm; khi
đó bỏ tick ở ô Công nghệ.

## Đăng ký vào bow-gid

Thêm vào `stacks` của `~/.bow-gid/registry.json` (bow chỉ tải repo có trong danh sách này, đúng tag đã ghim):

```json
{ "id": "mysql", "label": "MySQL", "repo": "github.com/Bow-T/bow-skill-mysql", "ref": "v1.0.0" }
```

Thử local trước khi đẩy GitHub: `"repo": "file:///đường/dẫn/tới/bow-skill-mysql"`.

## Phát hành phiên bản mới

Sửa qua PR → tăng `version` trong `bow-skill.json` → gắn tag (`git tag v1.0.1 && git push --tags`) → cập
nhật `ref` trong registry. bow tải theo tag và cache theo `<id>@<ref>`, nên sửa nội dung mà giữ nguyên tag sẽ
KHÔNG được tải lại.

**Đừng sửa tay file trong `skills/`** — sửa `tools/build.mjs` (thêm một `patch()` có nhãn) rồi chạy lại,
không thì lần cập nhật từ nguồn kế tiếp sẽ ghi đè mất. Cách cập nhật: cuối `THIRD_PARTY.md`.
