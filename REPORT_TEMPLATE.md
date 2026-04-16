# CSC4005 – Lab 2 Report

## 1. Thông tin chung
- Họ và tên: Nguyễn Văn Tiến
- Lớp: KHMT 17-01
- Repo: FIT-DNU-CS-16-01/fit-dnu-cs-16-01-17-01-csc4005-csc4005_lab2_cnn_neu-csc4005_lab2_neu_cnn_starter
- W&B project: csc4005-lab2-neu-cnn

## 2. Bài toán
Mục tiêu của bài lab là xây dựng mô hình phân loại ảnh lỗi bề mặt thép trên tập NEU-CLS với 6 lớp:
Crazing, Inclusion, Patches, Pitted_Surface, Rolled-in_Scale, Scratches.

Yêu cầu so sánh:
- MLP baseline (Lab 1)
- CNN train from scratch
- Transfer learning (ResNet18/MobileNet/VGG11_BN)

Tiêu chí đánh giá:
- Best Validation Accuracy
- Test Accuracy
- Epoch time
- Số lượng tham số trainable

## 3. Mô hình và cấu hình
### 3.1. MLP baseline từ Lab 1
- Model: [điền tên model MLP từ Lab 1]
- Cấu hình chính: [learning rate, batch size, số epoch, regularization]
- Ghi chú: Lấy kết quả từ Lab 1 để đối chiếu với CNN/transfer learning.

### 3.2. CNN from scratch
- Model: CNN-small
- Lệnh chạy (tham khảo):

```bash
python -m src.train --data_dir [duong_dan_du_lieu] --run_name cnn_small_baseline --model_name cnn_small --train_mode scratch --optimizer adamw --lr 0.001 --weight_decay 0.0001 --dropout 0.3 --epochs 20 --batch_size 32 --img_size 64 --patience 5 --augment --use_wandb
```

- Cấu hình đã dùng thực tế: đúng theo lệnh trên.

### 3.3. Transfer learning
- Model: ResNet18
- Train mode: transfer và finetune
- Lệnh chạy (tham khảo):

```bash
python -m src.train --data_dir [duong_dan_du_lieu] --run_name resnet18_transfer --model_name resnet18 --train_mode transfer --optimizer adamw --lr 0.001 --weight_decay 0.0001 --dropout 0.3 --epochs 10 --batch_size 32 --img_size 128 --patience 3 --augment --use_wandb

python -m src.train --data_dir [duong_dan_du_lieu] --run_name resnet18_finetune --model_name resnet18 --train_mode finetune --optimizer adamw --lr 0.0001 --weight_decay 0.0001 --dropout 0.3 --epochs 10 --batch_size 16 --img_size 128 --patience 3 --augment --use_wandb
```

- Cấu hình đã dùng thực tế: đúng theo 2 lệnh trên.

## 4. Bảng kết quả
| Model | Train mode | Best Val Acc | Test Acc | Epoch time | Trainable Params | Nhận xét |
|---|---|---:|---:|---:|---:|---|
| MLP | scratch | [điền từ Lab 1] | [điền từ Lab 1] | [điền từ Lab 1] | [điền từ Lab 1] | Baseline để so sánh |
| CNN-small | scratch | 0.9556 | 0.9556 | 3.8635 s/epoch | 32,614 | Nhanh, nhẹ, độ chính xác tốt |
| ResNet18 | transfer | 0.9667 | 0.9630 | 16.7370 s/epoch | 3,078 | Chính xác nhỉnh hơn scratch, trainable params rất ít |
| ResNet18 | finetune | 1.0000 | 1.0000 | 40.1102 s/epoch | 11,179,590 | Chính xác cao nhất nhưng tốn thời gian/tài nguyên |

Nguồn số liệu:
- `outputs/<run_name>/metrics.json`
- `outputs/<run_name>/history.csv`
- W&B run summary

## 5. Phân tích learning curves
- Chèn hình `curves.png` của từng run nếu có.
- Nhận xét:
- `cnn_small_baseline`: hội tụ nhanh, epoch time thấp, độ ổn định tốt.
- `resnet18_transfer`: val/test acc nhỉnh hơn CNN-small, phù hợp khi cần cải thiện chất lượng mà vẫn giữ chi phí huấn luyện vừa phải.
- `resnet18_finetune`: đạt kết quả cao nhất, nhưng thời gian mỗi epoch lớn nhất.
- Không thấy dấu hiệu overfitting rõ ở các run hiện tại (đặc biệt finetune đạt tốt cả val và test).

## 6. Confusion matrix và lỗi dự đoán sai
- Chèn `confusion_matrix.png` từ từng run.
- Một số cặp lớp dễ nhầm (dựa trên metrics hiện có):
- `Inclusion` và `Pitted_Surface` (xuất hiện ở cả scratch và transfer).
- `Scratches` và `Inclusion` (mức nhỏ, chủ yếu ở transfer).
- Nguyên nhân khả dĩ:
- Hình thái bề mặt tương tự nhau ở một số mẫu.
- Ảnh grayscale và resize nhỏ có thể làm mất chi tiết phân biệt.
- Khi finetune toàn bộ backbone, mô hình học đặc trưng tốt hơn nên giảm gần như toàn bộ nhầm lẫn.

## 7. Kết luận
- CNN có cải thiện so với MLP không?
- Có xu hướng cải thiện, nhưng cần điền số liệu MLP Lab 1 để kết luận định lượng cuối cùng.
- Transfer learning có tốt hơn không?
- Có. `ResNet18-transfer` nhỉnh hơn `CNN-small` về độ chính xác. `ResNet18-finetune` cho kết quả tốt nhất.
- Khi nào nên chọn transfer learning thay vì train from scratch?
- Nên chọn transfer learning khi dữ liệu không quá lớn và cần đạt hiệu năng tốt nhanh.
- Nên chọn finetune khi có đủ tài nguyên tính toán và cần tối đa hóa độ chính xác.
