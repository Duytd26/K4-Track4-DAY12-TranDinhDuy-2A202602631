# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | Trần Đình Duy |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `2A202602631` |
| GitHub | `Duytd26` |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | `3778` / `7871`, `7872`, `7873`, `7874`, `7875` |
| Số seed huấn luyện bạn dùng (`--seeds`) | `5` |

Lệnh tái lập toàn bộ trên local: `python run_all.py --student-id 2A202602631` (mặc định 5 seed).
Bản nộp dùng bằng chứng của lần chạy đầy đủ cuối cùng với 167.276 cửa sổ train, 71.857 cửa sổ test và 2.883 quỹ đạo giữ lại; không dùng chế độ `--quick`.

## Kết quả cuối

Số liệu từ `evidence/closedloop.json`, `openloop.json`, `gate.json`: L2 là trung bình ± độ lệch chuẩn qua 5 seed; va chạm là dải min–max qua seed. Hai baseline xác định chỉ có một lần đánh giá. Độ lệch chuẩn L2 mô tả biến thiên huấn luyện, không phải khoảng tin cậy.

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1,2497 | 20,7423 | BLOCK |
| IDM | 1,5298 | 0,0694 | (baseline) |
| EgoMLP | 0,7579 ± 0,0036 | 22,7541–26,4308 | BLOCK |
| PercMLP | 0,6194 ± 0,0024 | 0,8672–2,3933 | BLOCK |
| PercMLP+ (sau khi sửa) | 0,8144 ± 0,0140 | 0,2081 (cả 5 seed) | RELEASE trong phạm vi lab |

## Bảng đối chiếu thứ hạng — dùng cho Câu 4

L2 và va chạm dưới đây là trung bình qua các seed, theo `evidence/closedloop.json["ranking"]`.

| Bộ điều khiển | L2 trung bình (m) | Hạng vòng hở | Va chạm trung bình (%) | Hạng vòng kín |
|---|---:|---:|---:|---:|
| constant-velocity | 1,2497 | 3 | 20,7423 | 3 |
| IDM | 1,5298 | 4 | 0,0694 | 1 |
| EgoMLP | 0,7579 | 2 | 25,2029 | 4 |
| PercMLP | 0,6194 | 1 | 1,5609 | 2 |

## Bảng tác nhân phản ứng — dùng cho Câu 6

Theo `evidence/agents.json["rear_comparison_t_react_0.6"]`, mỗi mô hình học lấy lần chạy xấu nhất theo tỷ lệ va chạm bất kỳ trong thế giới phản ứng; so sánh hai thế giới trên cùng lần chạy đó.

| Bộ điều khiển | Va chạm bất kỳ: xe sau phát lại (%) | Va chạm bất kỳ: xe sau phản ứng (%) | Va chạm phía sau: phát lại / phản ứng (%) |
|---|---:|---:|---:|
| constant-velocity | 73,3704 | 73,3704 | 0 / 0 |
| IDM | 70,0741 | 5,1481 | 64,9259 / 0 |
| EgoMLP | 71,1481 | 71,1481 | 0 / 0 |
| PercMLP | 74,4815 | 20,4444 | 55,8519 / 0 |
| PercMLP+ | 69,7037 | 5,2963 | 64,4074 / 0 |

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

Theo `evidence/openloop.json` và `evidence/closedloop.json`, bảng đối chiếu phía trên dùng L2 và tỷ lệ va chạm trung bình qua 5 seed; IDM đạt 1,5298 m / 0,0694%, còn PercMLP đạt 0,6194 m / 1,5609%.
Các cặp đảo thứ tự là constant-velocity/IDM, constant-velocity/EgoMLP, IDM/EgoMLP và IDM/PercMLP; IDM chuyển từ hạng 4 vòng hở lên hạng 1 vòng kín.
L2 chấm độ giống quỹ đạo người lái; IDM giả định xe dẫn giữ tốc độ khi dự báo nên có thể khác người lái, nhưng phản hồi từ khoảng cách và tốc độ tương đối giúp tránh va chạm khi tự lái.
EgoMLP chỉ thấy lịch sử tốc độ của chính xe: L2 0,7579 m thấp hơn 1,2497 m của constant-velocity nhưng không nhận biết xe dẫn hay khoảng cách, nên lỗi phanh và sai số tích luỹ đẩy va chạm trung bình lên 25,2029%.
Lát cắt xe dẫn sắp phanh chiếm 4,4463% của 71.857 cửa sổ test; L2 PercMLP trên lát cắt là 0,8968 m, cao hơn 0,6194 m toàn bộ, cho thấy điểm gộp che tình huống khó.
L2 qua các seed của EgoMLP nằm trong 0,7551–0,7649 m, của PercMLP trong 0,6167–0,6232 m; dải vòng hở nhỏ không ngăn tỷ lệ va chạm vòng kín dao động đáng kể.

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

Theo `evidence/closedloop.json`, trên cùng 2.883 quỹ đạo test, EgoMLP va chạm lần lượt 721, 656, 762, 746 và 748 lần ở các seed 7871–7875.
Dải tỷ lệ qua 5 seed là 22,7541–26,4308%, rộng 3,6767 điểm phần trăm, vượt giới hạn ổn định 1 điểm phần trăm của R5; trung bình qua seed là 25,2029%.
Khoảng Wilson 95% của seed tốt nhất 7872 là [21,2605%; 24,3201%], còn seed xấu nhất 7873 là [24,8533%; 28,0711%]; hai khoảng không chồng nhau.
Wilson thể hiện độ bất định khi ước lượng tỷ lệ va chạm từ một tập hữu hạn ở mỗi seed, còn dải qua seed thể hiện độ nhạy với quá trình huấn luyện; hai nguồn biến thiên này cần báo riêng.
Nếu chỉ báo seed 7872, người đọc thấy 22,7541% và bỏ sót lần chạy 26,4308%; chọn seed thuận lợi sẽ che tính thiếu ổn định, nên cổng dùng seed xấu nhất cùng cận trên Wilson.

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

Theo `evidence/agents.json`, chọn `T = 1,4 s` vì có khoảng cách Wasserstein-1 nhỏ nhất giữa phân phối time-headway mô phỏng và người lái.
IDM hiệu chỉnh có W1 = 0,3396 s, so với 0,8418 s của xe không phản ứng; trung vị time-headway người lái là 1,8473 s, nên tác nhân hiệu chỉnh khớp phân phối quan sát tốt hơn.
Bảng tác nhân phía trên xét lưới 2.700 ô ở độ trễ xe thử 0,6 s, xe sau cách 16 m và có trễ phản ứng 0,6 s; dùng lần chạy xấu nhất của mỗi mô hình theo va chạm bất kỳ trong thế giới phản ứng.
Xe sau phát lại giữ tốc độ cũ bất kể xe thử phanh, nên có thể đâm vào đuôi xe IDM đang tránh xe trước; khi biết phản ứng, phần va chạm phía sau 64,9259% biến mất và va chạm bất kỳ của IDM còn 5,1481% ở vùng bất khả thi phía trước.
IDM vẫn tốt hơn constant-velocity trong cả hai thế giới, nhưng thế giới phát lại làm lợi thế trông nhỏ hơn rất nhiều; với PercMLP, thứ tự so với constant-velocity đảo từ 74,4815% > 73,3704% sang 20,4444% < 73,3704%.
Hiệu chỉnh bằng time-headway cải thiện độ giống một phân phối quan sát được; điều đó chưa chứng minh tác nhân mô phỏng phản ứng đúng với mọi tài xế hoặc mọi tình huống ngoài dữ liệu này.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

Theo `evidence/gate.json`, PercMLP bị BLOCK bởi R2, R3, R4, R5 và R6: cận trên va chạm log 3,0178% > giới hạn 1,0694%; cận trên lát cắt phanh 16,4508% > 5,5525%; diện tích va chạm còn tránh được 15,2963% > 2%; va chạm với xe sau phản ứng 20,4444% > 7,1481%.
R1 và R7 đạt trước khi sửa; R5 trượt vì dải va chạm log 0,8672–2,3933% rộng 1,5262 điểm phần trăm, vượt ngưỡng 1 điểm phần trăm qua 5 seed.
R4 và R6 cho thấy mô hình chưa xử lý tốt họ kịch bản cắt đầu ngoài log; R2 và R3 phản ánh sai số khi tự lái trên các trạng thái khác người lái, còn R5 cho thấy kết luận nhạy với seed huấn luyện.
Bổ sung 20.000 kịch bản cắt đầu để học phản ứng phanh mạnh từ IDM, rồi chạy 2 vòng DAgger: ghi trạng thái chính mô hình ghé qua trên log huấn luyện, gắn nhãn dự báo 3 s của IDM ở trạng thái đó và huấn luyện lại; các cấu hình được lưu trong `evidence/fix.json`.
Sau sửa, cả 7 quy tắc đều đạt: R2 có cận trên 0,4533%; R3 có cận trên 3,0627%; R4 còn 0,1481%; R6 còn 5,2963%; cả 5 seed cùng có tỷ lệ va chạm log 0,2081%, nên R2–R6 chuyển từ trượt sang đạt và dải R5 còn 0 điểm phần trăm.
L2 vòng hở tăng từ 0,6194 lên 0,8144 m trong khi an toàn vòng kín cải thiện; PercMLP+ vẫn va chạm 0,2081% so với IDM 0,0694%, nên kết quả hỗ trợ qua cổng không-kém-hơn với biên cho phép, chưa chứng minh tốt hơn IDM.
Quyết định là RELEASE trong phạm vi kiểm thử của lab; ở tốc độ chưa thấy 20 m/s, diện tích va chạm còn tránh được của PercMLP+ vẫn dao động 0,0370–3,6296% qua seed, trong khi IDM là 0% (`evidence/fix.json`), nên đề xuất mở rộng kiểm thử ngoài miền và chạy shadow mode có giám sát trước khi cân nhắc triển khai thực tế.

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

Mô phỏng chỉ xét chuyển động dọc và điều khiển với bước thời gian 0,1 s, không kiểm tra toàn bộ hệ thống nhận thức, chuyển làn hay tương tác ở nút giao; cần bổ sung mô phỏng không gian, kiểm thử cảm biến và thử nghiệm xe có giám sát.
Xe dẫn trong log không phản ứng theo xe thử; xe sau IDM được hiệu chỉnh W1 = 0,3396 s chỉ giống phân phối time-headway, chưa đại diện hành vi đa dạng của người lái, nên cần dữ liệu tương tác và nhiều mô hình tác nhân được kiểm chứng độc lập.
614 sự kiện phanh và 2.700 ô lưới tại tốc độ cruise 10 m/s vẫn chỉ phủ một họ kịch bản: khoảng cách 2,7601–59,3123 m và tốc độ tiến gần 0–10 m/s; ở 20 m/s, PercMLP+ còn tới 3,6296% ô va chạm tránh được theo `evidence/fix.json`, nên cần mở rộng tốc độ, tình huống hiếm và điều kiện đường trước khi quyết định phạm vi vận hành.
Thử nhiễu khoảng cách 1 m và tốc độ xe dẫn 0,5 m/s trong `evidence/closedloop.json` chỉ là một giả định nhiễu; phải đo phân phối lỗi, mất tín hiệu và độ trễ trên hệ thống thật rồi kiểm thử lại.
Năm seed huấn luyện và 2.883 quỹ đạo test từ cùng nguồn chưa chứng minh độ tin cậy ở sự kiện cực hiếm hoặc miền dữ liệu khác; Wilson chỉ phản ánh bất định tỷ lệ trong giả định lấy mẫu, nên cần tập test độc lập, thêm seed và phân tích phụ thuộc giữa các quỹ đạo.
PercMLP+ học nhãn IDM nên qua cổng không đồng nghĩa vượt chuyên gia; đề xuất giới hạn phạm vi vận hành, so với IDM trên bộ kịch bản chưa dùng để thiết kế cách sửa, rồi chạy shadow mode và kiểm thử có giám sát trước mọi quyết định triển khai thực tế.
