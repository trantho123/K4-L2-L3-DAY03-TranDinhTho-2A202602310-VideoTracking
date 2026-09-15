# Báo cáo Ngày 3 — Tracking Annotation

Họ tên: `Trần Đình Thọ`
Ngày: `15/9/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | 16 phút |
| Thời gian gán `clip_01` | 27 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | 8 |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che hoặc xuất hiện rất nhỏ: giữ cùng ID khi vẫn nhận ra cùng xe, chỉ bắt đầu track khi xác định được đó là xe bốn bánh.
2. Xe rời khung: đặt biên `outside` đúng frame cuối còn nhìn thấy xe, tránh để bbox treo sau khi xe đã ra khỏi ảnh.
3. Bbox thay đổi khi xe đổi hướng hoặc bị che: dùng keyframe dày hơn ở đoạn chuyển động mạnh và chỉ khoanh phần nhìn thấy.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: kiểm tra tính liên tục của ID và các đoạn xe bị che; không phát hiện ID switch.
- Lượt 2: kiểm tra frame bắt đầu/kết thúc; diagnostics sau đó còn phát hiện ID 4 dư ở frame 149–151.
- Lượt 3: kiểm tra bbox ở giữa track; các điểm cần xem lại là frame 9, 96 và 164.

Hình thức làm bài: **cá nhân**, không làm nhóm nên không có reviewer partner và không có lượt kiểm chéo giữa hai người.
Số lỗi trong bản của người khác / số lỗi người khác tìm trong bản của tôi: **Không áp dụng**.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Vì làm cá nhân nên không có ca hai người quyết khác nhau. Qua diagnostics, luật cần viết rõ hơn là: bbox phải kết thúc đúng frame cuối xe còn nhìn thấy; xe rời khung rồi quay lại dùng track mới; keyframe phải dày ở các đoạn xe đổi hướng hoặc bị che.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `3ee6e7ef0058d58837b708449c8e28881fd21ed350279e40981f89e82e3ee568` |
| Thời điểm khóa | `2026-09-15T08:39:10.461998+00:00` |
| Số row / frame / track trước khi mở reference | `521 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.820 | 0.793 | 0.854 | 0.906 | 0.943 | 0.892 | 0.895 | 5 | 57 | 0 |
| Sau rework / bản nộp hiện tại | 0.820 | 0.793 | 0.854 | 0.906 | 0.943 | 0.892 | 0.895 | 5 | 57 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo | 149-151 | 4 | Diagnostics ghi nhận cần đặt `outside` đúng frame xe rời khung; bản nộp hiện tại vẫn là bản đã khóa trước đó. |
| Bbox trôi | 9 | 2 | Diagnostics ghi nhận cần thêm hoặc điều chỉnh keyframe quanh frame 9; chưa có bản rework riêng để đối chiếu. |
| Thiếu đoạn / bbox trôi | 96, 164 | 5, 8 | Diagnostics ghi nhận cần kiểm tra độ phủ track và keyframe; chưa có bản rework riêng để đối chiếu. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | Python 3.13.15 / ultralytics 8.4.145 / torch 2.11.0+cpu / lap 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` và `botsort-reid.yaml` |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / `[2, 5, 7]` |
| device | CPU; `persist=True`; 190 frame |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.820 | 0.793 | 0.854 | 0.906 | 0.943 | 0.892 | 0.895 | 5 | 57 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.756 | 0.699 | 0.819 | 0.887 | 0.883 | 0.741 | 0.878 | 126 | 9 | 0 |

## 5. Phân tích — năm câu hỏi
                     HOTA    DetA    AssA    LocA    IDF1    MOTA    MOTP      FP      FN    IDSW
------------------------------------------------------------------------------------------------
ban_vs_gold         0.820   0.793   0.854   0.906   0.943   0.892   0.895       5      57       0
bytetrack_vs_gold   0.709   0.649   0.776   0.846   0.875   0.749   0.823      88      54       2
reid_vs_gold        0.763   0.711   0.820   0.872   0.900   0.792   0.860      91      26       2
reid_vs_ban         0.756   0.699   0.819   0.887   0.883   0.741   0.878     126       9       0

Cổng annotation: ĐẠT {'IDF1': 0.943, 'MOTA': 0.892, 'MOTP': 0.896}


**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi là `0.892`, thấp hơn IDF1 `0.943` khoảng `0.051`. IDF1 tập trung vào chất lượng ghép identity giữa prediction và ground truth, còn MOTA cộng nhiều loại lỗi gồm FN, FP và IDSW. MOTA không phạt nặng lỗi ID vì IDSW chỉ là một thành phần và bị đặt trong tổng số object-ground-truth detections; nếu detector vẫn tìm đúng vị trí và ID chỉ sai ở ít frame, MOTA có thể vẫn cao trong khi IDF1 giảm rõ hơn.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So với ByteTrack, ReID tăng IDF1 từ `0.875` lên `0.900` và AssA từ `0.776` lên `0.820`, nhưng IDSW vẫn là `2`. ReID cũng giảm FN từ `54` xuống `26`, dù FP tăng nhẹ từ `88` lên `91`. Trong sequence quanh frame `106-116`, cả hai tracker đều có các track dư gần vùng xe chồng/che: ByteTrack có ID `10` và `41`, còn ReID có ID `7`, `27`; vì vậy ReID cải thiện độ phủ nhưng không loại bỏ lỗi identity và false positive. Các ID switch cụ thể cũng khác vị trí: ByteTrack ở frame `59` và `94`, ReID ở frame `87` và `113`. Đây là so sánh hai hệ thống tracker, không phải causal ablation cô lập riêng tác động của ReID, vì implementation association của ByteTrack và BoT-SORT cũng khác nhau. Thí nghiệm appearance threshold cho thấy `0.7` và `0.8` giống hệt nhau (`IDF1 0.900`, `IDSW 2`); ở `0.9`, IDF1 giảm rất nhẹ xuống `0.899` và FN tăng lên `27`.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

ByteTrack có DetA `0.649`, FP `88`, FN `54`; ReID có DetA `0.711`, FP `91`, FN `26`. Như vậy ReID bắt được nhiều đoạn xe hơn nhưng đồng thời tạo thêm một số bbox thừa. Vì cùng detector input, phần chênh lệch về FP/FN còn chịu ảnh hưởng association và cách tracker giữ track, không thể quy toàn bộ cho detector. Với ByteTrack, FN và các đoạn bị mất cho thấy vấn đề nổi bật là độ phủ detection/track; với ReID, FP tăng và các ID như `7`, `27`, `38` không khớp gold cho thấy vẫn có lỗi association hoặc giữ nhầm track.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Một chỗ annotation đúng hơn ReID là vùng frame `16-116`: ReID tạo ID `7` không khớp track gold nào trong `43` frame, trong khi annotation có 8 track và chỉ có `5` FP. Đây là bằng chứng model giữ thêm track không có đối tượng tương ứng, nên không sửa annotation chỉ vì model có bbox.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Frame `164`, track ID `8`, là nơi cần xem lại annotation: diagnostics của annotation báo IoU chỉ `0.525`, còn ReID cũng báo bbox lệch ở frame `164` với IoU `0.59`. Ngoài ra annotation chỉ phủ track gold 8 trong `23/33` frame và track gold 6 trong `43/56` frame. Vì vậy cần xem lại keyframe và đoạn bắt đầu/kết thúc của các track này, nhưng chưa có evidence để kết luận phải đổi ID.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Tôi sẽ bổ sung vào `GUIDELINE_MINI.md` các quy tắc định lượng: đặt `outside` ở frame cuối còn nhìn thấy xe; giữ ID khi xe bị che dưới 25 frame; xe rời khung rồi quay lại dùng ID mới; bbox chỉ ôm phần nhìn thấy; và đặt keyframe dày tại các điểm xe đổi hướng, chồng lấp hoặc xuất hiện/rời khung. Quy trình mới là tự kiểm ba lượt theo ID, biên track và bbox giữa track, sau đó chạy validator và khóa pre-gold trước khi xem gold/model.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` 
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md` — không áp dụng vì làm cá nhân; không tạo file reviewer giả
- [x] `reports/REPORT.md`
