# Báo cáo Day 08 — 2A202604055 - Phạm Quang Huy

> Một file duy nhất cho cả bài. Mọi con số kèm **mẫu số** và nguồn. Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`yolo26m.pt`) | 0.3 (mặc định) cho cả 6 lớp | `threshold_sweep.md`: person tối đa recall 76% (34 vật gold), motorbike 87% (15), car 88% (8): không ngưỡng nào ≤ 0.5 đạt 90%; bicycle (4), bus (0), truck (1) dưới 5 vật nên không chọn ngưỡng riêng. r01 (conf 0.25) so với r02 (conf 0.5): 215 → 139 box, ngưỡng 0.5 bớt box thừa nhưng làm mất xe máy và người ở xa (img_007, img_030). Nhận xét: 0.3 nằm giữa hai lần chạy: bớt box thừa so với 0.25 nhưng vẫn giữ các xe và người ở xa có điểm 0.40–0.49 mà 0.5 làm mất. Vì recall trên gold vẫn dưới 90% ở mọi ngưỡng, khi duyệt phải chủ động tìm vật sót chứ không chỉ sửa box máy. |
| three_wheeler | YOLOE-26s (`yoloe-26s-seg.pt`), prompt: `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.3 (mặc định) | `threshold_sweep.md`: tối đa recall 88% (8 vật gold). r03: YOLOE tìm đúng tuk-tuk / xích lô mà YOLO26 gọi sai lớp hoặc bỏ sót (img_002 0.93, img_032 0.92, img_011 0.28). Nhận xét: YOLOE là lựa chọn duy nhất cho lớp này vì YOLO26 (COCO) không có lớp xe ba bánh. Tuy vậy, ở ngưỡng 0.3, box tuk-tuk 0.28 ở img_011 vẫn bị loại, còn các box thừa 0.37–0.40 trên xe máy và bán tải ở img_002 vẫn lọt vào; vì vậy `three_wheeler` là lớp cần soi kỹ nhất khi duyệt. |

- Lệnh `merge_prelabels.py` đã dùng:
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
  (không truyền `--thr`, tương đương chuỗi `--thr ""` mà `threshold_sweep.py` in ra; YOLOE đứng trước nên thắng khi hai box chồng IoU ≥ 0.6.)
- **Phiên bản SAM bạn chọn** và lý do: SAM 2.1 base (`sam2.1_b.pt`). Dữ liệu so sánh trên img_013 (r05, r07, r09, r10): cùng 3 click, SAM 2.1 dính người lái còn MobileSAM tách được phần lớn; khi thêm `--box`, hai bản cho mask gần như nhau (37.5% so với 35.0% diện tích ảnh); thời gian một lần chạy trên CPU 7.1 s (SAM 2.1) so với 4.5 s (MobileSAM). Chọn SAM 2.1 base vì trong pipeline SAM luôn nhận box từ YOLO / YOLOE (r04), và ở điều kiện có box hai bản cho kết quả tương đương; chênh 2.6 s mỗi lần chạy không đáng kể khi chỉ tách mask cho 3 vật; đây cũng là bản mặc định của lab và đã chạy ổn định từ Part 2 (r04: 16/16 box có mask). Cách đặt prompt ảnh hưởng đến mask nhiều hơn phiên bản model: thêm `--box` tách được người lái, còn thêm 4 click âm làm hỏng mask (r06). Giới hạn: chỉ so sánh trên 1 ảnh; nếu máy yếu hơn hoặc cần tách mask hàng loạt, MobileSAM là lựa chọn hợp lý.
- Lỗi nào **không sửa được bằng đổi ngưỡng**? Sửa thế nào? Sai lớp có conf cao: img_011 tuk-tuk → `truck` 0.77, img_013 tuk-tuk → `car` 0.42, img_032 xích lô → `bicycle` 0.70, img_002 bán tải → `truck` 0.73 (r01, r02, r04). Trên gold, trong 15 vật máy bỏ sót ở ngưỡng 0.10: 7 vật không có box nào, 7 vật bị gán lớp khác, 1 box lỏng. Cách xử lý đã dùng: YOLOE-26s cho `three_wheeler`, gộp trước YOLO26 để thắng box car/truck; phần còn lại sửa tay khi duyệt. Nhận xét: đây là lỗi về **lớp** và về **việc có box hay không**, không phải lỗi về **độ tin cậy**. Hạ ngưỡng không tạo ra box cho 7 vật máy không thấy; nâng ngưỡng cũng không bỏ được box sai lớp có conf 0.73–0.77 mà không mất theo các box đúng có cùng mức điểm. Cách sửa hiệu quả là đổi model hoặc prompt cho đúng lớp (YOLOE cho xe ba bánh) và thêm bước người duyệt tìm vật sót. Hiệu quả vẫn chưa trọn: sau khi gộp, three_wheeler vẫn chỉ có 3/24 box máy được giữ nguyên và phải vẽ thêm 8 (`review_stats.md`).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng**: điểm cao mà sai: img_011 `truck` 0.77 (thực là tuk-tuk), img_002 `truck` 0.73 (bán tải, theo guideline là `car`), img_032 `bicycle` 0.70 (xích lô). Điểm thấp mà đúng: img_011 tuk-tuk YOLOE 0.28; img_007 xe máy ở xa 0.40 và 0.49.
- Giới hạn: chọn ngưỡng trên chính gold nên số đo máy trên gold là lạc quan: ngưỡng được chọn và chấm trên cùng 4 ảnh, 70 đối tượng; ba lớp có dưới 5 vật nên không có căn cứ chọn ngưỡng riêng; khoảng tin cậy 95% của precision / recall máy trên gold là 51–73% (`review_stats.md`). Nhận xét: vì cuối cùng không lớp nào được chọn ngưỡng riêng (cả 7 lớp dùng 0.3 mặc định), mức lạc quan do "chọn trên chính gold" là nhỏ; giới hạn lớn hơn là cỡ mẫu: 70 đối tượng trên 4 ảnh cho khoảng tin cậy rộng 22 điểm phần trăm, và 3/7 lớp gần như không được đo. Vì vậy các con số trên gold chỉ nên đọc như ước lượng thô. Muốn chọn ngưỡng có căn cứ hơn cần một tập riêng để chọn, giữ gold chỉ để đo.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 190 | 37.8% | 503 box máy |
| edited | 139 | 27.6% | 503 box máy |
| deleted | 163 | 32.4% | 503 box máy |
| added | 63 | 16.1% | 392 đối tượng sau duyệt |

Đã loại 27 đối tượng quanh vùng bẫy khỏi bảng trên. Trong deleted + added có 12 trường hợp đổi lớp, nên số vật máy bỏ sót thật ≈ 63 − 12 = 51.

Theo lớp (accepted / edited / deleted / added): person 103 / 66 / 90 / 21; motorbike 52 / 51 / 35 / 21; car 28 / 11 / 15 / 5; three_wheeler 3 / 9 / 12 / 8; bicycle 2 / 1 / 6 / 2; bus 2 / 0 / 1 / 4; truck 0 / 1 / 4 / 2.

- Lớp nào máy tệ nhất? Vì sao? `three_wheeler`: chỉ 3/24 box máy = 12.5% được giữ nguyên (9 sửa, 12 xoá) và phải vẽ thêm 8 vật. Nguyên nhân: YOLO26 không có lớp này nên gọi xe ba bánh là car / truck / bicycle (img_011, img_013, img_032); YOLOE bù được nhưng box hay lỏng hoặc bao nhiều xe (img_002 box rộng hơn xe, img_032 một box bao nhiều xích lô đỗ sát nhau) và bỏ sót xích lô đỗ (img_007). `truck` có 0/5 accepted nhưng mẫu quá nhỏ để kết luận. Về số tuyệt đối, `person` bị xoá nhiều nhất: 90/259 box = 34.7%, chủ yếu là box nhỏ ở xa và box trùng.
- So với gold: precision / recall của **máy** 44/70 = 62.9% / 44/70 = 62.9% (TP 44, FP 26, FN 26; KTC 95%: 51–73%) (mẫu số: 70 đối tượng gold)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12 = 83.3% (KTC 95%: 55–95%); loại nào lọt: 2/3 bẫy `loose` (img_022, img_031), các loại `extra`, `wrong_class`, `missing` bắt đủ 3/3; vì sao? Cả hai bẫy lọt đều là `loose` được tạo từ một box máy vốn đã sai: ở img_022 box gốc bao chung người đội nón và bao tải, ở img_031 box gốc nằm trên một chiếc mũ cam, không phải người. Khi duyệt, tôi thu box img_022 về ôm sát người và xoá box img_031. Theo hình học thì đó là sửa đúng, nhưng script chỉ tính là bắt được khi box cuối còn IoU ≥ 0.75 với box gốc của máy, nên hai bẫy này bị tính là lọt. Như vậy đây không phải do bỏ bước kiểm "ôm sát" của checklist mà do bẫy nằm chồng lên lỗi máy; tôi vẫn ghi lại hai ca này (CAL-05) và không sửa nhãn để "lấy điểm".
- Accepted cao mà bắt bẫy thấp hoặc ngược lại nói gì về cách bạn duyệt? Accepted thấp (190/503 = 37.8%) đi cùng bắt bẫy đạt (10/12 = 83.3%) cho thấy tôi không chấp nhận box máy một cách thụ động với các lỗi máy **đã đưa ra** (box thừa, sai lớp, box lỏng). Thiên kiến tự động hoá xuất hiện ở dạng khác: QC vòng 1 có 16/18 lỗi là bỏ sót, đúng ở những vật máy **không** đề xuất box. Khi duyệt theo danh sách box máy, sự chú ý bị neo vào những gì máy đã vẽ, nên vật máy bỏ qua thì người duyệt cũng dễ bỏ qua.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 15/4 = 3.75 phút/ảnh vs duyệt 194/24 ≈ 8.1 phút/ảnh (số phút duyệt ước lượng từ mốc thời gian tạo task → export lần đầu, gồm cả lúc nghỉ và hỏi AI). Thiên lệch: Gán gold trước lẽ ra giúp duyệt nhanh hơn vì đã quen cảnh và quy tắc, vậy mà duyệt vẫn chậm hơn gán tay khoảng 2.2 lần. Mật độ vật hai bên tương đương (gold 72/4 = 18 vật/ảnh, batch 392/24 ≈ 16.3 vật/ảnh), nên khác biệt không đến từ độ khó ảnh. Phần lớn chênh lệch có thể do số phút duyệt gồm cả lúc nghỉ, lúc hỏi AI và lần đầu dùng CVAT để duyệt; số đo này vì vậy thiên về phía bất lợi cho pre-label và chỉ nên đọc như cận trên.
- Kết luận: chưa đáng dùng cho mọi lớp ở cấu hình hiện tại; hữu ích một phần cho `car`, không đáng cho `three_wheeler`, dựa trên: thời gian 8.1 so với 3.75 phút/ảnh (chưa thấy tiết kiệm); accepted chung 37.8% (< 50%, sửa một box thường không nhanh hơn vẽ mới); theo lớp, `car` 28/54 = 51.9% accepted, `person` 103/259 = 39.8%, `motorbike` 52/138 = 37.7%, `three_wheeler` 3/24 = 12.5%; vẫn phải vẽ thêm 63/392 = 16.1% đối tượng.
- **Ba quy tắc giảm thiên kiến của bạn**:
  1. **Đếm trước, so sau:** trước khi xem box máy, đếm nhanh số người và xe rõ ràng trong ảnh; sau khi duyệt, nếu số đối tượng lệch quá 2 so với số đếm thì quét lại ảnh. Kiểm tra bằng số: tỉ lệ `bo_sot` trong QC (vòng 1 là 16/156 đối tượng).
  2. **Quét lưới tìm vật sót (CAL-07):** sau khi duyệt hết box máy, quét ảnh theo lưới 3 × 3 để tìm vật cao ≥ 20 px chưa có box; cụm ≥ 5 vật nhỏ hơn thì vẽ `ignore_region`. Kiểm tra bằng việc làm: mỗi ảnh có một lượt quét riêng, và số vật `added` tìm được trong lượt này.
  3. **Không tin điểm cao:** mọi box có conf ≥ 0.7 thuộc lớp xe phải được kiểm lại lớp và 4 cạnh (lệch ≤ 2 px), vì lỗi nặng nhất có điểm cao (img_011 0.77, img_002 0.73, img_032 0.70). Kiểm tra bằng số: số box conf ≥ 0.7 bị đổi lớp hoặc sửa, so với tổng box conf ≥ 0.7.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | SAM 2.1 base | 2 dương, 1 âm | Có, gồm gần hết người lái và cả người ngồi phía sau | Click âm chỉ khoét vài lỗ trên lưng áo người lái; lem nền nhìn qua khung xe hở (45.4% ảnh) | `tuktuk_img013_sam2.jpg` |
| img_013 | SAM 2.1 base | 2 dương, 4 âm | Không | Mask co lại chỉ còn mái xe (10.8% ảnh, 2 polygon), mất thân xe | `tuktuk_img013_sam2_neg4.jpg` |
| img_013 | SAM 2.1 base | `--box` + 1 dương, 2 âm | Không, tách sạch người lái và người phía sau | Đầu xe phía phải (x ≈ 820–880) bị lỗ chỗ (37.5% ảnh) | `tuktuk_img013_sam2_box.jpg` |
| img_013 | MobileSAM | 2 dương, 1 âm | Còn dính mặt và dép người lái | Sàn và thân trước phía phải lỗ chỗ, 2 polygon (35.2% ảnh) | `tuktuk_img013_sam1.jpg` |
| img_013 | MobileSAM | `--box` + 1 dương, 2 âm | Không | Đầu xe phía phải lỗ chỗ như SAM 2.1 (35.0% ảnh) | `tuktuk_img013_sam1_box.jpg` |
| img_004 | SAM 2.1 base | 2 dương, 1 âm | Không có người lái trong xe; không dính người đứng cạnh | Lem một dải bóng đổ dưới gầm do click (700, 560) sát mặt đường (13.3% ảnh) | `tuktuk_img004_sam2.jpg` |
| img_017 | SAM 2.1 base | 2 dương, 1 âm | Không thấy người lái; không dính tuk-tuk phía sau | Click âm (645, 355) rơi lên mái xe (12.1% ảnh) | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc giúp được: 4.2.3 (bóng đổ không thuộc vật) chỉ ra lỗi lem bóng dưới gầm ở img_004; 4.2.2 (lỗ lớn bỏ khỏi polygon) áp dụng cho khoảng nhìn xuyên qua lan can tuk-tuk ở img_013. Quy tắc còn thiếu: 4.2 chưa nói rõ người lái hoặc khách ngồi trong xe phải tách khỏi mask xe (chỉ suy ra từ mục 2); câu kính xe còn mở ở v1.0 (đã chốt ở CAL-04); guideline không có hướng dẫn về cách đặt prompt, trong khi kết quả cho thấy cho SAM khung trước rồi mới thêm click âm tốt hơn hẳn việc chỉ thêm click âm (r06 so với r07).
- Interactor SAM trong CVAT: không thử.

## 6. Calibration (Part 6)

- Số quyết định: 7 (`calibration_decisions.csv`: CAL-01 → CAL-07; nguồn: 2 `loi_may`, 3 `cau_hoi_mo`, 1 `can_review`, 1 `bay_lot`); quan trọng nhất: CAL-07 (thêm bước quét tìm vật sót), vì bỏ sót là loại lỗi lớn nhất (16/18 lỗi QC vòng 1), có ở mọi lớp, và không sửa được bằng đổi model hay ngưỡng mà chỉ sửa được bằng quy trình duyệt.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu 2 (xe đỗ dày gần camera gán từng xe, CAL-03), câu 3 (hàng người mang không thuộc box `person`, CAL-05), câu 5 (kính thuộc mask xe, CAL-04), câu 7 (không đặt mức conf tự chấp nhận, CAL-02); thêm ca xe ba bánh bị máy gọi sai lớp (CAL-01), xe hàng rong 3 bánh (CAL-06), bước quét tìm vật sót sau khi duyệt (CAL-07).
- Nguyên nhân gốc: chủ yếu do **máy** (sai lớp xe ba bánh ở CAL-01, bỏ sót vật nhỏ ở CAL-07) cộng với **cách duyệt** bị neo vào box máy (CAL-07); **guideline mơ hồ** chỉ ở một số ca (xe hàng rong CAL-06; các câu 2, 3, 5, 7 của mục 6). CAL-05 xuất phát từ bẫy đặt trên một box máy vốn đã sai.

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh (3 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: Claude (AI), theo yêu cầu của tôi, sau đó tôi xem lại (**hạn chế**: không phải người kiểm độc lập; vòng 2 cùng một người vừa sửa vừa kiểm).
- Vòng 1 (`qc_report_auto_round1.md`): lỗi theo **đối tượng** 18/156 = 11.5% · theo **ảnh** 6/8 = 75.0% (ngưỡng Release ≤ 2%) → gợi ý Rework. Đã sửa trên CVAT: xoá 2 box, thêm 21 box và 1 `ignore_region` ở 6 ảnh (`labels/qc_fixed.json`), build lại release (24 ảnh, 430 đối tượng).
- Vòng 2 (`qc_report_auto.md`): lỗi theo **đối tượng** 0/175 = 0.0% · theo **ảnh** 0/8 = 0.0%.
- Top lỗi (vòng 1) + một ảnh ví dụ mỗi loại: bỏ sót 16/18 (img_018: người và xe máy nhỏ ở quảng trường; img_021: người lái giữa đoàn xe; img_035: người ngồi vỉa hè; img_003: xe đạp người dắt); thừa 2/18 (img_008 #83 trùng #81; img_006 #71 bao chung hai người đã có box riêng). Không ghi nhận sai lớp, sai thuộc tính, sai mask.
- **Giới hạn mẫu nhỏ**: vòng 1 cận trên Wilson 93%; vòng 2 dù 0/8 ảnh lỗi, cận trên vẫn là 32%. Seed 42 rút lại đúng 8 ảnh vừa sửa, nên vòng 2 chỉ xác nhận phần đã sửa; 16 ảnh ngoài mẫu chưa được quét lại. Nhận xét: kết quả vòng 2 chỉ chứng minh 6 ảnh đã được sửa đúng, không chứng minh tỉ lệ lỗi của cả lô ≤ 2%. Vì lỗi vòng 1 có tính hệ thống (bỏ sót trong cảnh đông), 16 ảnh chưa quét lại nhiều khả năng còn lỗi cùng loại.
- Script kiểm được / không kiểm được: Script kiểm được định dạng COCO, tên lớp, box nằm trong ảnh, đủ 24 ảnh, bốn trạng thái accepted / edited / deleted / added, tỉ lệ bắt bẫy, precision / recall so với gold, và tỉ lệ lỗi tính từ phiếu QC. Script không kiểm được: box có ôm sát vật thật không; lớp có đúng thực tế không; vật mà cả máy lẫn người duyệt cùng bỏ sót ở ngoài 4 ảnh gold (chính là loại lỗi chính của vòng 1); và người duyệt có thật sự làm theo checklist không. Phiếu QC cũng chỉ đúng bằng mắt người kiểm.

**Quyết định: Release**, ngày 2026-11-18. Lý do: **Hold?** Không: các câu hỏi guideline gặp trong lô đã được chốt ở v1.1 (CAL-01 → CAL-07) và không có ảnh hỏng (`--guideline-gap 0`, `--data-problem 0`). **Rework?** Vòng 1 có 6/8 ảnh lỗi > 2% và lỗi bỏ sót lặp lại, nên đã Rework: sửa 6 ảnh và thêm quy tắc CAL-07. Sau Rework, vòng 2 có 0/8 ảnh lỗi ≤ 2%, bắt bẫy 10/12 = 83.3% ≥ 80%, trong mẫu không còn lỗi hệ thống. **Release**, kèm cảnh báo: mẫu nhỏ (cận trên Wilson 32%) và 16 ảnh chưa quét lại; trước khi dùng lô này để huấn luyện nên áp dụng bước quét CAL-07 cho 16 ảnh đó.
Nếu Rework / Hold, việc cần làm tiếp: không áp dụng (Release).

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Có, dùng Claude (Claude Code) 
  - Cách tôi kiểm tra:
    - **Nhãn trên CVAT:** mở lại từng ảnh Claude đã sửa (img_022, img_031 và 6 ảnh QC: img_003, img_006, img_008, img_018, img_021, img_035) trong Job batch-review để xem box mới có ôm sát vật và đúng lớp không; đối chiếu tổng số box trên CVAT với file export (518 box sau khi import; 430 đối tượng trong release sau Rework).
    - **Nhận xét về model:** yêu cầu soát lại r01 và r02 lần thứ hai, rồi tự mở ảnh viz trong `work/viz/` để đối chiếu các lỗi được nêu trong `model_log.csv` (ví dụ tuk-tuk img_011 bị gán `truck` 0.77).
    - **Số liệu trong báo cáo:** đối chiếu từng con số và mẫu số với file nguồn (`review_stats.md`, `threshold_sweep.md`, `qc_report_auto*.md`, `time_log.csv`); chạy `validate_labels.py --all` và `check_submission.py`.
    - **Phần nhận xét do AI soạn:** đọc lại từng đoạn và chỉ giữ những ý khớp với ảnh và số liệu mình đã thấy.
- Gold có bị nhiễm không? Không nhìn pre-label trước khi gán xong gold. Trong lúc gán có hỏi AI về cách hiểu quy tắc (`conf` / `can_review`, vật bị che, `ignore_region`) nhưng không đưa ảnh gold cho AI xem hay nhờ AI gán.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11 Pro, không có GPU, PyTorch 2.15.0+cpu trong `.venv`; CVAT v2.81.0 chạy Docker. Ban đầu định dùng Colab, sau chuyển về chạy local CPU.
- Dùng cache `data/cache/` thay vì tự chạy? Không.
- Lỗi / bug script đã gặp (nếu có): chạy `prepare_workdirs.py` sai thư mục (phải chạy từ gốc repo); import COCO vào CVAT báo `Label 'motorbike' is not registered` vì Task #24 tạo ngoài Project, tạo lại Task #25 trong Project thì nạp đủ 518 box.
- **Ca không chắc quy tắc ở gold (box tick `can_review`)**: 4/72 box trong `submission/labels/gold_manual.json` (số liệu từ file export CVAT). Cột "Lý do" do tôi tự điền theo guideline v1.0.

  | Ảnh | Box (id CVAT) | Lớp | Vị trí `[x, y, w, h]` (px) | Lý do không chắc quy tắc |
  |---|---|---|---|---|
  | img_001 | #13 | bicycle | [794, 404, 14, 20] | Vật nhỏ, ảnh mờ nên không nhận diện rõ (nhìn không chắc là xe đạp). |
  | img_001 | #17 | person | [706, 390, 14, 36] | Ảnh mờ, không nhận diện rõ vật (nhìn không chắc là người). |
  | img_001 | #27 | motorbike | [523, 406, 19, 31] | Ảnh mờ, không nhận diện rõ vật (nhìn không chắc là xe máy). |
  | img_020 | #54 | truck | [948, 135, 76, 92] | Dựa vào ảnh không xác định được là `bus` hay `truck`; tôi chọn `truck` và tick `can_review` (guideline mục 4.3). |

  `validate_labels.py` cũng cảnh báo hai box cao dưới hoặc bằng 20 px: img_001 #13 (cao 20.0 px) và img_036 #59 motorbike (cao 19.2 px, không tick `can_review`).
- Một điều bạn làm khác lần sau: Bấm đồng hồ thật khi duyệt (ghi cả dòng `break`) thay vì ước lượng sau từ mốc thời gian, và quét lưới tìm vật sót ngay trong lần duyệt đầu thay vì để QC phát hiện, vì đó chính là nơi 16/18 lỗi vòng 1 xuất hiện.
