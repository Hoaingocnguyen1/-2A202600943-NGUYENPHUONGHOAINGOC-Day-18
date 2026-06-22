# Reflection — Day 18 Lakehouse Lab

**Anti-pattern team tôi dễ vướng nhất: "Small-file problem" (ingest streaming thành hàng nghìn file tí hon).**

Pipeline LLM-observability của team nhận log theo từng request gần như real-time. Cách tự nhiên nhất — mỗi micro-batch ghi một append — chính là cái NB2 tái hiện: 200 lần append → 200 file nhỏ, point-query mất ~584 ms vì engine phải mở metadata của mọi file. Đây là anti-pattern nguy hiểm vì nó *không gây lỗi*, chỉ âm thầm làm chậm và đội chi phí khi bảng lớn dần, nên dễ bị bỏ qua tới khi dashboard giật.

Lab cho thấy cách chữa rất rõ: `OPTIMIZE compact()` gộp 200 → 55 file (giảm latency còn ~103 ms, **speedup 5.7×**), và `Z-ORDER(user_id)` co cụm dữ liệu theo khóa truy vấn để file-skipping bằng min/max stats hoạt động — chỉ **1/55 file** chứa `user_id=4242` (**files-pruned 55×**). Bài học: tách *ghi nhanh* (Bronze append) khỏi *đọc nhanh* (compaction định kỳ + Z-order trên cột filter chính), và lên lịch OPTIMIZE như một bước bảo trì bắt buộc, không phải tùy chọn.

(~190 từ)
