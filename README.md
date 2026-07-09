
---

# Dự đoán khả năng tái nhập viện của bệnh nhân tiểu đường

## Tổng quan đề tài

Bài toán
Tái nhập viện xảy ra khi một bệnh nhân đã được xuất viện quay trở lại bệnh viện trong một khoảng thời gian nhất định. Tỷ lệ tái nhập viện cao thường phản ánh chất lượng điều trị hoặc chăm sóc sau xuất viện chưa tốt, đồng thời làm gia tăng chi phí y tế.

Mục tiêu
Sử dụng bộ dữ liệu hồ sơ bệnh án để trả lời các câu hỏi sau:
Các yếu tố nào ảnh hưởng mạnh nhất đến khả năng tái nhập viện của bệnh nhân đái tháo đường?
Có thể dự đoán nguy cơ tái nhập viện với độ chính xác cao khi chỉ sử dụng một số lượng đặc trưng hạn chế hay không?


### Các biến chính
|          *Tên biến*         |                 *Mô tả*                 |        *Kiểu dữ liệu / Giá trị* |
|-----------------------------|-----------------------------------------|------------------------------------------------------------
| Encounter ID                | Mã định danh duy nhất của mỗi lần khám  | Numeric                                                   |
| Patient Number              | Mã định danh duy nhất của bệnh nhân     | Numeric                                                   |
| Race                        | Chủng tộc của bệnh nhân                 | Asian, American, Afcica, Hispanic, Caucasian, Other       |
| Gender                      | Giới tính của bệnh nhân                 | Male, Female, Unknown/Invalid                             |
| Age                         | Nhóm tuổi của bệnh nhân                 | 10-year intervals (gộp nhóm:[0-10), [10-20), …, [90-100)) |
| Weight                      | Cân nặng của bệnh nhân                  | Pounds (Numeric)                                          |
| Admission Type              | Loại nhập viện                          | 9 values (Emergency, Urgent, Elective)                    |
| Discharge Disposition       | Tình trạng xuất viện                    | 29 values ( Home, Expired)                                |
| Admission Source            | Nguồn nhập viện                         | 21 values (Physician Referral, Emergency Room)            |
| Time in Hospital            | Số ngày nằm viện                        | Integer                                                   |
| Payer Code                  | Mã đơn vị thanh toán bảo hiểm           | 23 values (Self-Pay, Medicare)                            |
| Medical Specialty           | Chuyên khoa của bác sĩ điều trị         | 84 values (Cardiology, Internal Medicine)                 |
| Number of Lab Procedures    | Số lượng xét nghiệm được thực hiện      | Numeric                                                   |
| Number of Procedures        | Số lượng thủ thuật điều trị             | Numeric                                                   |
| Number of Medications       | Số lượng thuốc được kê                  | Numeric                                                   |
| Number of Outpatient Visits | Số lần khám ngoại trú trong năm trước   | Numeric                                                   |
| Number of Emergency Visits  | Số lần cấp cứu trong năm trước          | Numeric                                                   |
| Number of Inpatient Visits  | Số lần điều trị nội trú trong năm trước | Numeric                                                   |
| Diagnosis 1                 | Chẩn đoán chính                         | ICD9 first 3 digits (848 values)                          |
| Diagnosis 2                 | Chẩn đoán phụ thứ nhất                  | ICD9 first 3 digits (923 values)                          |
| Diagnosis 3                 | Chẩn đoán phụ thứ hai                   | ICD9 first 3 digits (954 values)                          |
| Number of Diagnoses         | Tổng số chẩn đoán                       | Numeric                                                   |
| Glucose Serum Test Result   | Kết quả xét nghiệm đường huyết          | >200, >300, Normal, None                                  |
| A1c Test Result             | Kết quả xét nghiệm HbA1c                | >8%, >7%, Normal, None                                    |
| Change of Medications       | Thay đổi thuốc điều trị đái tháo đường  | Change, No Change                                         |
| Diabetes Medications        | Có sử dụng thuốc điều trị đái tháo đường| Yes, No                                                   |
| Medication Features (24)    | Trạng thái của 24 loại thuốc điều 
                                trị đái tháo đường (như: metformin, insulin)| Up, Down, Steady, No |
| Readmitted                  | Thời gian tái nhập viện                 | <30 days, >30 days, No                                    |


### Các bước thực hiện gồm:
1. **Khám phá dữ liệu**:
   - Phân tích tập dữ liệu nhằm để xác định các giá trị còn thiếu, phân phối và tình trạng mất cân bằng.
   - Trực quan hóa mối quan hệ giữa các đặc trưng (ví dụ: time in hospital, number of medications) và readmission bằng Seaborn and Matplotlib.

2. **Tạo các đặc trưng**:
   - Tạo các đặc trưng mới như service_utilization (sum of outpatient, emergency, and inpatient visits).
   - Nhóm các mã ICD-9 thành danh mục để dễ phân lọai (ví dụ: Diabetes, Circulatory).
   - Kế đến áp dụng phép chueyẻn đổi cho các đặc trưng có phân phối lệch.

3. **Modeling**:
   - **Lựa chọn đặc trưng**: Dùng phương pháp Recursive Feature Elimination (RFE) với RandomForest để chọn 30 đặc trưng quan trọng.
   - **Cân bằng dữ liệu**: Dùng ADASYN để xử lý mất cân bằng lớp (88% non-readmitted so với 12% readmitted).
   - **Các mô hình dùng để thử nghiệm gồm**:
     - Decision Tree
     - XGBoost
     - LightGBM
   - **Tinh chỉnh lại các tham số**: Dùng GridSearchCV với kiểm định chéo để giản hóa hiệu suất cho mô hình.
   - **Đánh giá chỉ số**: Tập trung vào recall để nhận diện cho trường hợp tái nhập viện, precision, F1-score, ROC-AUC, and PR-AUC.

4. **Model cuối cùng**:
   - Lựa chọn XGBoost với các siêu tham số đã được tinh chỉnh nhờ sự cân bằng giữa hiệu suất và khả năng diễn giải.
   - Điều chỉnh ngưỡng dự báo để ưu tiên độ nhạy recall ≥ 0.70 và cũng tối đa hóa cho độ chính xác.

5. **Trực quan hóa**:
   - Xuất file filepredict.csv để xem và trực quan hóa dữ liệu dựa trên bản đồ để dễ dàng xem xét và tương tác.

## Results
The final XGBoost model was evaluated on a test set (20% of data) with a focus on identifying patients at risk of readmission within 30 days.


### Các yêu tố để dự báo chính
Dựa trên mức độ quan trọng của các giá trị số của mô hình XGBoost những yếu tố giúp dự đoán khả năng cao tái nhập viên gồm:
1. **Number of Inpatient Visits**: Số lần nhập viện trước đó càng cao thì càng tỷ lệ thuận với nguy cơ tái nhập viện.
2. **Service Utilization**: Tổng số lần tiếp xúc với dịch vụ y tế như: nội trú, ngoại trú, cấp cứu sẽ phản ánh mức độ phức tạp trong tình trạng bệnh nhân.
3. **Time in Hospital**: Thời gian nằm bệnh viện dài sẽ cho biết là có tình trạng bệnh nghiêm trọng dẫn đến tăng khả năng tái nhập viện.
4. **Number of Medications**: Càng nhiều số lượng thuốc thì sẽ phản ánh phác đồ điều trị phức tạp.
5. **Change in Medications**: Việc liên tục chỉnh các loại thuốc đái tháo đường cho thấy tình trạng kiểm soát bệnh mất ổn định.

### Trực quan hóa
- **Time in Hospital vs. Readmission**: Bệnh nhân tái nhập viện có thời gian nằm viện dài hơn một chút.
- **Age vs. Readmission**: Các bệnh nhân lớn tuổi trong nhóm tuổi từ 80->90 có khả năng nhập viện cao hơn.
- **Service Utilization**: Những bệnh nhân có mức độ sử dụng dịch vụ cao hơn có nhiều khả năng phải tái nhập viện.

## Hướng dẫn set up và sử dụng
### Yêu cầu
- Python 3.9+
- Libraries: `pandas`, `numpy`, `scikit-learn`, `xgboost`, `flask`, `imblearn`, `seaborn`, `matplotlib`

### Installation
1. Clone :
   git clone https://github.com/Phihungzz/Dudoantainhapvien-diabetic.git
   cd Dudoantainhapvien-diabetic

2. Tải dataset từ [UCI Repository](https://archive.ics.uci.edu/ml/datasets/Diabetes+130-US+hospitals+for+years+1999-2008) và đặt trong folder với tên diabetic_data.csv.

3. Khởi động Jupyter Notebook bằng cmd:
   jupyter notebook

4. Mở file prediction-readmission.ipynb và chạy để thực hiện phân tích.

