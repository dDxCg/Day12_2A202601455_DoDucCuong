# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Đỗ Đức Cường |
| Mã học viên | 2A202601455 |
| Repo | https://github.com/dDxCg/Day12_2A202601455_DoDucCuong.git |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-2a202601455-doduccuong.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | render env |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | render redis add-on |
| `RATE_LIMIT_PER_MINUTE` | ✅ | render env |
| `MONTHLY_BUDGET_USD` | ✅ | render env |
| `LOG_LEVEL` | ✅ | render env |

## Lệnh Kiểm Tra
```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-2a202601455-doduccuong.onrender.com/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-2a202601455-doduccuong.onrender.com/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-2a202601455-doduccuong.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-2a202601455-doduccuong.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-2a202601455-doduccuong.onrender.com/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
# 1: /health
HTTP/2 200 
date: Mon, 10 Aug 2026 03:42:32 GMT
content-type: application/json
cf-cache-status: DYNAMIC
rndr-id: f63cdfa3-5586-49ca
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-ray: a28bfdbd3f9c8484-HKG
alt-svc: h3=":443"; ma=86400

# 2: /ready
HTTP/2 200 
date: Mon, 10 Aug 2026 03:43:59 GMT
content-type: application/json
rndr-id: a1a0516f-c4cf-444b
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
cf-ray: a28bffdb3e32ce39-SIN
alt-svc: h3=":443"; ma=86400

# 3: /ask without KEY
HTTP/2 401 
date: Mon, 10 Aug 2026 03:45:53 GMT
content-type: application/json
rndr-id: d5bd06c6-a81b-47f2
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
cf-ray: a28c02a0bf9afdf0-SIN
alt-svc: h3=":443"; ma=86400

# 4: /ask with KEY
HTTP/2 200 
date: Mon, 10 Aug 2026 03:48:27 GMT
content-type: application/json
cf-cache-status: DYNAMIC
rndr-id: 3ca60331-f77d-473d
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-ray: a28c066518f206fa-HKG
alt-svc: h3=":443"; ma=86400

{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud.","user_id":"sv-test","history_length":0,"cost_usd":2.145e-05,"tokens":{"in":3,"out":35}}

# 5: Rate limit check
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl


