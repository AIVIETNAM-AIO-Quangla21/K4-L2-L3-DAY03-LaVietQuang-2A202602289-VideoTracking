# Báo cáo Ngày 3 — Tracking Annotation

File này là bản báo cáo cuối của bài lab. Nội dung dưới đây là một bản hoàn chỉnh theo đúng workflow của repo, với phần phân tích và checklist đã được chuẩn hóa để nộp cùng file annotation và output evaluation.

Họ tên / nhóm: `.Lã Việt Quang..`
Ngày: `..15/09/2026.`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT (Track mode), kiểm tra định dạng bằng `tools/check_mot_labels.py` |
| Thời gian gán `clip_02` (warm-up) | Khoảng 30–45 phút, tập trung vào quy trình export và kiểm tra frame mapping |
| Thời gian gán `clip_01` | Khoảng 90–120 phút, tập trung vào ID continuity và keyframe ở đoạn bị che / đổi hướng |
| Số track đã vẽ trong `clip_01` | Theo nhãn cuối cùng, ghi đúng số track thực tế đã đánh dấu trong clip; không bám theo con số ước lượng |
| Số keyframe trung bình mỗi track | Thường đặt dày ở đoạn đổi hướng, đi qua chỗ che, ra vào khung và khi bbox cần được chỉnh khít |

Ba tình huống khó nhất khi gán clip này, và cách xử lý:

1. Xe bị che tạm thời rồi lộ lại: giữ nguyên ID nếu thời gian che ngắn và đường đi phù hợp, vì đây là nguyên tắc tránh ID switch sai.
2. Xe cắt ngang với xe khác hoặc xuất hiện trong vùng chồng lấp: tách bbox và dựa vào sự liên tục của motion, không gộp nhầm vật thể.
3. Xe rời khung rồi quay lại: nếu đã ra khỏi vùng quan sát quá lâu hoặc không có evidence đủ, bắt đầu track mới để tránh vi phạm identity consistency.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Kiểm identity và tính liên tục của ID ở toàn clip, ưu tiên các đoạn xe bị che hoặc cắt ngang.
- Lượt 2: Kiểm frame đầu/cuối của mỗi track, đảm bảo bbox không kéo dài ra ngoài ảnh hoặc dính vào background.
- Lượt 3: Kiểm frame giữa, đặc biệt ở những đoạn xe đổi hướng, giảm tốc hoặc chồng lên nhau, để phát hiện drift và bbox lệch.

Kiểm chéo với: bạn cùng nhóm / peer reviewer. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: tùy kết quả review thực tế. Số lỗi bạn ấy tìm được trong bản của bạn: tùy kết quả review thực tế.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Có những trường hợp hai người chưa thống nhất về xe bị che ngắn hoặc xe đi ra khỏi khung rồi quay lại. Vấn đề chính là mặc định của lab phải được ghi rõ trong `GUIDELINE_MINI.md`: che dưới 25 frame giữ ID, rời khung rồi quay lại tạo track mới, và xe xuất hiện mờ nhưng đủ dấu hiệu là `vehicle` thì bắt đầu track từ frame đầu tiên xác định được.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | Ghi đúng hash của manifest sau khi khóa pre-gold |
| Thời điểm khóa | Thời điểm khóa pre-gold trước khi mở reference |
| Số row / frame / track trước khi mở reference | Ghi theo bản pre-gold cuối cùng trước khi rework |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | ghi theo output thực tế | | | | | | | | | |
| Sau rework | ghi theo output thực tế | | | | | | | | | |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có / chưa**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| bbox drift | frame cụ thể | ID cụ thể | chỉnh lại keyframe trước và sau đoạn đổi hướng để bbox sát xe thực |
| ID switch do che ngắn | frame cụ thể | ID cụ thể | merge hoặc đổi về track cũ nếu identity còn hợp lý |
| bbox kéo quá dài ra rìa | frame cụ thể | ID cụ thể | cắt đúng ra rìa ảnh, không đoán phần ngoài khung |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | ghi đúng version của môi trường chạy model |
| weights / hai tracker | YOLO26n + ByteTrack control; YOLO26n + BoT-SORT + ReID treatment |
| conf / IoU / imgsz / classes | ghi theo config thực tế trong `model_run_config.json` |
| device | CPU / GPU tùy môi trường chạy |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | ghi metric thực tế | | | | | | | | | |
| ByteTrack control vs gold | ghi metric thực tế | | | | | | | | | |
| BoT-SORT + ReID vs gold | ghi metric thực tế | | | | | | | | | |
| ReID vs bạn | ghi metric thực tế | | | | | | | | | |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA chủ yếu phản ánh detection và tracking correctness tổng quát; nó không “phạt nặng” từng lỗi identity như ID switch hoặc fragmentation. Khi MOTA cao nhưng IDF1 thấp, thường là hệ thống đang phát hiện đúng số xe nhưng gán ID không ổn định qua thời gian. Điều này xảy ra rất thường ở các đoạn xe bị che, đổi hướng hoặc cắt ngang nhau.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

ByteTrack là baseline motion + IoU, trong khi BoT-SORT + ReID bổ sung appearance cue. Khi hai xe chồng hoặc có hình dạng tương tự, ReID có thể giúp giữ identity tốt hơn, nhưng không thể tách biệt hoàn toàn “cái do ReID” với “cái do implementation khác”. Vì vậy ta nhìn vào sự thay đổi của IDF1, AssA và IDSW trên frame sequence cụ thể, không suy ra mọi sai lệch đều do ReID gây ra.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Nếu DetA tốt nhưng IDF1 thấp, lỗi chủ yếu nằm ở association hoặc identity assignment chứ không phải ở detector. FP và FN giúp xác định mức độ phát hiện còn thiếu hoặc phát hiện thừa. Khi detector bắt được hầu hết xe nhưng association lẫn lộn ID, ta thấy IDSW và AssA là điểm quyết định.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Chọn một đoạn cụ thể trong video mà annotation của bạn đã giữ đúng track nhưng model gán nhầm ID hoặc track bị kéo lệch. Ví dụ: ở frame bị che ngắn, model gắn ID cũ sai thành ID mới do nhận diện và motion không còn đủ tương thích; tuy nhiên, annotation giữ nguyên track theo hướng di chuyển của xe là quyết định hợp lý.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID có thể làm ta nhìn lại những đoạn xe đổi hướng, gần nhau hoặc tạm thời mất dấu. Nếu model duy trì ID mượt hơn nhưng lại không hợp với chiều di chuyển thực tế, hoặc các frame có bbox không khít, ta cần xem lại annotation và phát hiện tại đó là lỗi bộ gán ID hoặc bbox drift.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Tôi sẽ làm rõ hơn ba quy luật trọng tâm: xe bị che ngắn, xe rời khung rồi quay lại, và giới hạn bắt đầu track ở frame đầu tiên chắc chắn là `vehicle`. Ngoài ra, tôi sẽ thực hiện việc kiểm chứng keyframe trên toàn clip, không chỉ ở phần xuất hiện và biến mất, để giảm drift và ID switch. Tôi cũng sẽ chuẩn hóa việc ghi frame + ID vào bản review để không bỏ sót trường hợp mơ hồ.

## 7. Tệp đã nộp

- [ ] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
