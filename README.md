# Time Series Project: Water Consumption Forecasting

## Giới thiệu

Dự án này phân tích và dự báo mức tiêu thụ nước theo thời gian (hourly time series) dựa trên dữ liệu thực tế của hệ thống cấp nước. Mục tiêu chính là hiểu xu hướng, tính mùa vụ và xây dựng mô hình forecasting để dự đoán mức tiêu thụ trong tương lai.

Dữ liệu được lưu trong file gốc:

- `waterconsumption_rawdata.csv`

Ngoài ra, dự án còn chứa các kết quả phân tích, hình ảnh EDA và mô hình ARIMA/SARIMAX đã huấn luyện.

## Cấu trúc thư mục

```text
Time_Series/
├── waterconsumption_rawdata.csv
├── All_Charts/                  # Biểu đồ EDA và kiểm định tính dừng
├── ARIMA_Results/               # Kết quả mô hình ARIMA
├── SARIMAX_Results/             # Kết quả mô hình SARIMAX
├── experiment/                  # Notebook thử nghiệm mô hình
├── README.md                    # Tài liệu hướng dẫn
└── ...
```

## Dataset

File dữ liệu gốc: `waterconsumption_rawdata.csv`

### Cấu trúc cột

| Cột | Mô tả |
| --- | --- |
| Date | Ngày quan sát |
| Time | Khung giờ theo từng giờ, ví dụ `00:00-01:00` |
| Consumption | Lượng tiêu thụ nước (đơn vị: liters hoặc unit phù hợp dữ liệu nguồn) |
| Temperature | Nhiệt độ không khí |
| Rain | Mức mưa |
| Sun | Chỉ số ánh nắng mặt trời |

### Ví dụ dữ liệu

```csv
Date,Time,Consumption,Temperature,Rain,Sun
1/1/2016, 00:00-01:00 ,1480.00,0.6,0,0
1/1/2016, 01:00-02:00 ,1362.00,0.6,0,0
1/1/2016, 02:00-03:00 ,1296.00,0.5,0,0
```

## Bản chất của dữ liệu

- Dữ liệu là dạng time series theo giờ.
- Có sự thay đổi theo ngày/giờ và khả năng xuất hiện mô hình mùa vụ theo chu kỳ 24 giờ.
- Các biến ngoại vi như `Temperature`, `Rain`, `Sun` có thể hỗ trợ mô hình forecasting bằng cách tích hợp yếu tố môi trường.

## Mục tiêu phân tích

1. Khám phá xu hướng và mô hình theo giờ, ngày và tuần.
2. Kiểm tra tính dừng của chuỗi thời gian.
3. Xây dựng mô hình ARIMA và SARIMAX để dự báo tiêu thụ nước.
4. So sánh hiệu suất của các mô hình dựa trên các metric đánh giá.

## Phân tích và kết quả

### 1. EDA (Exploratory Data Analysis)

Thư mục `All_Charts/` chứa các hình ảnh mô tả:

- `01_eda_timeline.png` - xu hướng thời gian
- `02_eda_histogram.png` - phân phối dữ liệu
- `03_eda_boxplot.png` - boxplot
- `04_eda_hourly_pattern.png` - mẫu theo giờ
- `05_eda_weekly_pattern.png` - mẫu theo tuần
- `06_stationarity_original.png` - kiểm định tính dừng trên chuỗi gốc
- `07_stationarity_diff1.png` - sau khi lấy sai phân bậc 1
- `08_acf_48.png`, `09_pacf_48.png` - hàm tự tương quan và tự tương quan từng phần
- `10_acf_168_seasonality.png` - mô hình mùa vụ theo 168 giờ
- `11_...` đến `20_...` - residuals và so sánh dự báo

### 2. Mô hình ARIMA

Thư mục `ARIMA_Results/` chứa:

- `best_model.pkl` - mô hình tốt nhất đã lưu
- `training_data.csv`, `validation_data.csv`, `testing_data.csv` - dữ liệu chia tập
- `forecast_results.csv` - kết quả dự báo
- `evaluation_metrics.csv` - các chỉ số đánh giá
- `forecast_vs_actual.png` - so sánh dự báo và giá trị thực
- `model_summary.txt` - summary mô hình

### 3. Mô hình SARIMAX

Thư mục `SARIMAX_Results/` chứa:

- `sarimax_model_summary.txt` - summary mô hình SARIMAX
- `model_performance_comparison.csv` - so sánh hiệu suất mô hình
- `comprehensive_forecast_results.csv` - kết quả dự báo toàn diện

## Mô hình sử dụng

Dự án đã thử nghiệm các mô hình phổ biến trong forecasting time series:

- ARIMA
- SARIMAX
- Kết hợp biến ngoại vi như `Temperature`, `Rain`, `Sun`

Dựa trên summary mô hình, SARIMAX cho thấy khả năng phù hợp tốt hơn với dữ liệu có tính mùa vụ theo chu kỳ 24 giờ.

## Cách sử dụng

### Đọc dữ liệu

```python
import pandas as pd

df = pd.read_csv('waterconsumption_rawdata.csv')
print(df.head())
```

### Chuyển đổi thời gian

Nếu cần, hãy chuyển `Date` và `Time` về dạng datetime index để phục vụ modeling:

```python
import pandas as pd

df = pd.read_csv('waterconsumption_rawdata.csv')
df['datetime'] = pd.to_datetime(df['Date'] + ' ' + df['Time'].str.split('-').str[0], format='%m/%d/%Y %H:%M')
df = df.set_index('datetime')
```

### Huấn luyện mô hình

```python
from statsmodels.tsa.statespace.sarimax import SARIMAX

model = SARIMAX(
    df['Consumption'],
    exog=df[['Temperature', 'Rain', 'Sun']],
    order=(2, 1, 1),
    seasonal_order=(1, 0, 1, 24),
    enforce_stationarity=False,
    enforce_invertibility=False,
)
result = model.fit(disp=False)
forecast = result.forecast(steps=24)
```

## Lưu ý quan trọng

- Dữ liệu thường tồn tại các yếu tố biến động theo giờ và theo mùa.
- Trước khi mô hình hóa, nên kiểm tra tính dừng và xử lý dữ liệu thiếu/không hợp lệ nếu có.
- Với chuỗi thời gian theo giờ, các chu kỳ như 24 giờ, 168 giờ có thể cần được xem xét kỹ lưỡng.

## Kết luận

Dự án này cho thấy khả năng áp dụng các kỹ thuật time series forecasting trong việc dự đoán lượng tiêu thụ nước. Từ EDA đến mô hình SARIMAX, toàn bộ quy trình đã tạo ra cơ sở để theo dõi, đánh giá và dự đoán nhu cầu sử dụng nước theo thời gian thực.

## Tài liệu tham khảo

- `statsmodels` - Time series modeling
- `pandas` - Data handling and preprocessing
- `matplotlib` / `seaborn` - visualization
- `ARIMA`, `SARIMAX` - forecasting techniques

## Tác giả / người thực hiện

Dự án được xây dựng cho mục tiêu phân tích time series và dự báo tiêu thụ nước.

---

Nếu muốn, tôi có thể tiếp tục viết thêm:

- phiên bản README ngắn gọn hơn cho GitHub,
- phiên bản tiếng Anh chuyên nghiệp,
- hoặc một file `requirements.txt` / notebook hướng dẫn chạy mô hình.
