# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | Trần Đình Duy |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `2A202602631` |
| GitHub | `Duytd26` |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | `3778` / `7871`, `7872`, `7873` |
| Số seed huấn luyện bạn dùng (`--seeds`) | `3` |

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json` (cho mỗi mô hình học, lấy **dải qua các seed**).

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | | | |
| IDM | | | (baseline) |
| EgoMLP | | | |
| PercMLP | | | |
| PercMLP+ (sau khi sửa) | | | |

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

Theo `evidence/data_audit.json`, dữ liệu có 992.679 dòng và 9.993 quỹ đạo ban đầu, lấy mẫu mỗi 0,1 s.
Tương quan giữa `Acc_LV` và đạo hàm tốc độ xe dẫn chỉ đạt 0,04535, trong khi giá trị tương ứng của xe theo sau là 0,92909; vì vậy không dùng `Acc_LV` để khai thác ca phanh.
Thay vào đó, dùng gia tốc suy ra bằng sai phân từ `Speed_LV` đã làm mượt bằng trung bình trượt 5 mẫu.
Loại 383 quỹ đạo trùng hệt nhau, còn 9.610 quỹ đạo sạch; nếu giữ bản trùng ở cả train và test thì kết quả có thể bị đánh giá quá lạc quan.
Chia theo quỹ đạo sau khi loại trùng để các khung gần nhau của cùng một chuyển động không xuất hiện ở cả hai tập; lần chạy này dùng seed chia 3778.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

Theo `evidence/replay.json`, phát lại cùng mã cho 0/600 đoạn khác nhau; bản refactor float32 khác giá trị chính xác ở 600/600 đoạn nhưng 0 đoạn vượt dung sai, sai lệch tuyệt đối lớn nhất là 9,3201e-5 m/s².
Dung sai là `|new - golden| <= 0,001 + 0,001 * |golden|` trên lệnh gia tốc; sai khác nhỏ do độ chính xác và thứ tự phép toán không đủ để coi là thay đổi hành vi ở mức dung sai đã chọn.
AEB làm lệch 8 đoạn: 125, 131, 160, 178, 234, 345, 412 và 476.
Kiểm tra trực tiếp đoạn 125 trong `work/replay_log.jsonl` cho thấy các khung 34–40 đổi lệnh: ở khung 34, khoảng cách 4,4608 m, tốc độ tiến gần 3,007 m/s, TTC = 1,4835 s < 1,5 s nên AEB đổi lệnh từ -5,3258 thành -6 m/s².
Phát nhanh nhất không có khung trễ hoặc rớt; phát theo thời gian thật có 586 khung trễ và 582 khung rớt trên 60.127 khung, khiến 378/600 đoạn vượt dung sai so với bản chuẩn.
Đây là chi phí xử lý được mô phỏng: khi quá hạn hoặc bộ điều khiển còn bận, hệ thống giữ lệnh trước đó; phát nhanh nhất bỏ qua ràng buộc thời gian nên che các lỗi này.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

Theo `evidence/scenarios.json`, lưới gồm 60 mức khoảng cách × 45 mức tốc độ tiến gần = 2.700 ô, với tốc độ cruise 10 m/s, khoảng cách 2,7601–59,3123 m và tốc độ tiến gần 0–10 m/s.
Biên bất khả thi là `gap < dv * t_react + dv² / (2 * 9)` khi `dv > 0`: ngay cả phanh tối đa cũng không đủ quãng đường để tránh va chạm.
Khi độ trễ tăng từ 0 lên 0,6 s, vùng bất khả thi tăng từ 0,9630% lên 5,1481%; diện tích va chạm của IDM tăng tương ứng 4,1852 điểm phần trăm.
IDM không gây va chạm ở vùng còn tránh được trong cả hai lưới; kiểm chứng phanh tối đa so với công thức cho 0 ô lệch ngoài dải sát biên ở cả hai độ trễ.
Giới hạn phanh người lái ở phân vị 99,9 là 3,6248 m/s², thấp hơn giới hạn cơ cấu 9 m/s²; 5,2222% ô đòi phanh mạnh hơn mức người lái này.
Khai thác được 614 sự kiện phanh trên 9.610 quỹ đạo, chiếm 6,3892%; log tự nhiên có các ca phanh nhưng chưa đại diện đủ cho những cảnh cắt đầu cần phanh rất mạnh, nên không thể chỉ chấm trên log rồi suy ra an toàn trên toàn lưới.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

(trả lời ở đây)

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

(trả lời ở đây)

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

(trả lời ở đây)

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

(trả lời ở đây)

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

(trả lời ở đây)
