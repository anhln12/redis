# redis

Câu lệnh tìm ra key redis đang chiếm ram:
```
redis-cli -a "pass_redis" --bigkeys
```

Việc chạy không ảnh hưởng đến hệ thống

Câu lệnh xóa 1 key có số lượng member lớn
```
[71.30%] Biggest set    found so far '"v.smscore.model.PDUTransfer"' with 155637882 members
UNLINK vn.smscore.model.PDUTransfer
```

Khác với DEL (xóa đồng bộ lập tức, gây treo Redis nếu key quá lớn), lệnh UNLINK sẽ ngắt liên kết của key ngay lập tức và giao việc giải phóng bộ nhớ RAM cho một luồng nền (background thread) xử lý ngầm một cách từ từ không ảnh hưởng đến hệ thống

