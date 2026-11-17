# Giải thích chi tiết: Hệ thống bia quang điện tử đo phân bố đầu đạn (Chu et al., Photonics 2019)

Nov 6, 2026 · @bình

## 1. Vấn đề bài báo muốn giải quyết

Bài báo của Chu và cộng sự (*Photonics*, 2019, tập 6, trang 126) đề xuất một bia điện tử không tiếp xúc, không cần thấu kính, đo được tọa độ đầu đạn với sai số dưới ±2 mm.

Khi thử nghiệm, nghiệm thu một loại súng hay đạn, người ta cần biết **các viên đạn rơi vào đâu trên mặt phẳng bia**, tức độ tản mát (độ chụm). Hiện có hai cách đo:

- **Đo tiếp xúc:** dùng bia giấy, lá nhôm hoặc tấm thép. Cách này đơn giản và rẻ, nhưng mỗi lần bắn lại phải thay bia. Bia dùng một lần nên khó tự động hóa và độ lặp lại kém.
- **Đo không tiếp xúc:** tạo một "màn sáng" song song bằng laser hoặc LED chiếu vào một dãy cảm biến. Đầu đạn bay qua sẽ che sáng, và cảm biến nào bị che thì cho biết vị trí. Nhược điểm của cách này:
  - Phải đặt làm thấu kính Fresnel hoặc bộ thấu kính lồi để biến chùm sáng phân kỳ thành chùm song song. Thấu kính này đắt.
  - Độ chính xác phụ thuộc rất nhiều vào độ song song của màn sáng, nên căn chỉnh quang học khó và mất công.
  - Khung cơ khí giữ thấu kính chiếm chỗ, làm diện tích đo hữu ích nhỏ đi.

**Mục tiêu của bài:** làm một hệ đo không tiếp xúc nhưng **không cần thấu kính chuẩn trực**. Hệ thống phải rẻ, gọn, dễ lắp và vẫn chính xác.

## 2. Nguyên lý hoạt động

Hệ thống gồm 5 khối:

1. **Dãy LED:** nguồn sáng tạo màn sáng.
2. **Hai lớp mảng khe hở** (double-layered aperture arrays): hai tấm có khoan các hàng lỗ nhỏ, đặt chồng lên nhau, có tác dụng "chia nhỏ" và lọc màn sáng.
3. **Dãy photodiode (PD):** mỗi PD đặt ngay dưới một khe. Vì vậy **mỗi PD ứng đúng một vị trí tọa độ**.
4. **Mạch ngưỡng thích nghi:** khuếch đại tín hiệu và quyết định thế nào thì tính là có đạn bay qua.
5. **Bộ thu thập dữ liệu (FPGA) và máy tính:** ghi lại PD nào phản hồi rồi tính ra tọa độ.

Khi đầu đạn bay qua, lượng sáng tới một số PD giảm đi một chút. Mạch điện biến sự sụt giảm đó thành một xung số. Tọa độ của các PD phát ra xung chính là tọa độ điểm đạn bay qua. Ghép hai màn sáng vuông góc với nhau (một cho trục X, một cho trục Y) sẽ được tọa độ 2D.

## 3. Phân tích toán học

Hai lớp khe cách nhau 181 mm và PD đặt cách nhau 4 mm khiến mỗi PD chỉ "nhìn thấy" đúng phần ánh sáng ở vị trí của nó.

### Vấn đề gốc: LED có góc phân kỳ

Ánh sáng LED không song song mà tỏa ra thành hình nón. Không có gì che chắn thì một PD nhận ánh sáng từ **rất nhiều LED** ở nhiều góc khác nhau. Khi đó đạn bay qua một chỗ sẽ làm nhiều PD cùng phản hồi và tọa độ bị nhòe.

Nếu không có khe, bề rộng vùng LED chiếu tới được một PD là:

```latex
l_2 = 2(h_1 + h_2)\tan\alpha + l_1
```

trong đó α là nửa góc phân kỳ của LED, l₁ là đường kính mặt nhạy của PD, h₁ và h₂ là các khoảng cách theo phương truyền sáng.

### Bước 1: thêm một lớp khe

Đặt mảng khe 1 phía trên dãy PD để giới hạn thị trường (field of view) của từng PD. Với màn sáng dài h₁ = 1,5 m, PD đường kính l₁ = 3 mm và khe cách PD h₂ = 200 mm:

- Thị trường của một PD rộng l₃ = 50 mm.
- Bề rộng chùm LED chiếu tới vùng PD vẫn là l₄ = 9,8 mm, còn quá rộng.

Thêm vào đó, **ánh sáng xiên đi qua các khe bên cạnh** vẫn lọt tới PD, nên một lớp khe là chưa đủ.

### Bước 2: thêm lớp khe thứ hai

Đặt mảng khe 2 nằm giữa khe 1 và dãy PD. Lớp này làm hai việc:

- Thu hẹp thêm bề rộng chùm sáng tới PD.
- **Chặn tia xiên nhất** từ khe lân cận, để nó không chạm tới PD.

Từ quan hệ hình học (tam giác đồng dạng), các tác giả lập được:

- **Phương trình (2):** bề rộng chùm sáng mới l₄′, tính theo h₁, h₂ và h₄.
- **Phương trình (3):** quan hệ giữa khoảng cách hai lớp khe h₄ và khoảng cách giữa hai PD d:

```latex
h_4 = h_2 - \frac{d - l_1}{\tan\alpha}
```

Như vậy muốn biết đặt hai lớp khe cách nhau bao nhiêu thì trước hết phải chọn khoảng cách d giữa hai PD.

### Bước 3: chọn khoảng cách giữa các PD (d)

Lấy ví dụ đầu đạn đường kính 5,8 mm. Khi bay qua, đạn có thể kích hoạt vài PD liền nhau, và tọa độ được lấy là **tâm của nhóm PD đó**. Trường hợp xấu nhất là mép đạn sát đến PD kế bên nhưng chưa đủ để kích hoạt nó, lúc đó sai số lớn nhất.

Đặt điều kiện sai số ≤ 2 mm (phương trình 4) thì được **d ≤ khoảng 4,2 mm**. Tác giả chọn **d = 4 mm**. Thay vào các phương trình (2) và (3):

- Hai lớp khe cách nhau khoảng **181 mm**.
- Bề rộng chùm sáng tới mỗi PD chỉ còn **l₃ ≈ 3,58 mm**, xấp xỉ đúng đường kính PD (3 mm).

Kết luận của phần này: **phản hồi của PD tương ứng một-một với tọa độ trên màn sáng**.

## 4. Mô phỏng quang học

Mô phỏng Zemax xác nhận hai lớp khe tạo ra các đốm sáng tách biệt cho từng PD, và cho ra ngưỡng phát hiện bằng 1/5 điện áp đỉnh.

### 4.1. Mô hình Zemax

Các tác giả mô phỏng hệ thống bằng phần mềm Zemax (chế độ non-sequential) với các thông số:

| Thành phần | Thông số |
| --- | --- |
| Dãy LED | Nửa góc phân kỳ 3°, LED cách nhau 4 mm, dãy dài 1600 mm |
| Hai lớp khe | Cách LED 1500 mm; khe đường kính 3 mm, cách nhau 4 mm; hai lớp cách nhau 181 mm |
| Dãy PD | Cách LED 1700 mm; mỗi PD đường kính 3 mm |

**Kết quả (Hình 5, 6):** trên mặt PD xuất hiện một **mảng các đốm sáng tách biệt**, mỗi đốm ứng với một PD. So với lúc chưa có khe, cường độ sáng tới các PD **đồng đều hơn hẳn**, nhờ vậy độ chính xác đo tăng lên.

**Đánh đổi:** khe chặn bớt ánh sáng nên công suất tới PD giảm, tức là **độ nhạy phát hiện giảm**. Đó là lý do cần phần phân tích độ nhạy tiếp theo.

### 4.2. Phân tích độ nhạy

Tín hiệu điện áp do đạn tạo ra là:

```latex
V(t) = \Delta\varphi(t) \cdot \varepsilon \cdot \gamma \cdot R
```

- Δφ(t): mức sụt công suất quang khi đạn che sáng
- ε: độ nhạy chuyển đổi quang-điện của PD
- γ: hệ số khuếch đại của mạch
- R: điện trở chuyển dòng thành áp

Trong cùng một hệ thống, ε, γ và R cố định, nên **biên độ tín hiệu chỉ phụ thuộc vào lượng sáng bị che**. Muốn phát hiện được đạn ở mọi vị trí trên màn sáng thì V(t) phải luôn lớn hơn ngưỡng.

**Mô phỏng:** khi không có đạn, PD nhận công suất P₁ ≈ 1,82×10⁻⁴ W. Các tác giả cho đạn 5,8 mm đi qua ở các khoảng cách 100, 300, 600, 900, 1200 và 1400 mm tính từ LED, và ở nhiều vị trí lệch ngang khác nhau. Kết quả (Hình 7, 8, 9):

1. Khi đạn **thẳng hàng với PD** (Y = 0), công suất giảm mạnh nhất. Lệch sang hai bên thì mức giảm tụt nhanh.
2. Đạn **càng gần PD** (càng xa LED) thì mức sụt công suất càng lớn. Ngược lại, đạn bay gần phía LED thì khó phát hiện hơn.
3. Ở **hai đầu dãy** độ nhạy thấp do hiệu ứng biên. Vì vậy dãy thiết bị nên dài hơn vùng đo hữu ích một đoạn.

Từ đây các tác giả chọn **ngưỡng phát hiện = 1/5 điện áp đỉnh**: hễ công suất tới PD sụt đi 1/5 là tính có đạn. Ngưỡng này bảo đảm phát hiện được đạn ở mọi vị trí trong vùng đo, với sai số trong khoảng ±2 mm.

## 5. Mạch ngưỡng thích nghi

Mạch ngưỡng thích nghi biến mức sụt sáng 1/5 thành một xung số sạch, tránh báo sai. Muốn phát hiện tin cậy thì cần ba thứ: PD phản hồi nhanh, mạch ít nhiễu và quyết định chính xác.

**Tại sao cần ngưỡng "thích nghi"?** Mỗi PD nhận một mức sáng nền hơi khác nhau, và mức nền có thể trôi theo nhiệt độ, độ già của LED hay ánh sáng môi trường. Một ngưỡng cố định có thể khiến PD này báo nhầm còn PD kia bỏ sót.

**Cách làm:**

- Tín hiệu PD được khuếch đại, sau đó đưa vào một mạch RC (tụ điện và điện trở) để tạo **ngưỡng bám theo mức đỉnh** của chính PD đó.
- Vì tụ nạp và xả chậm, trong khoảnh khắc ngắn ngủi đạn bay qua thì ngưỡng **gần như không đổi**.
- Bộ so sánh (comparator):
  - Khi tín hiệu tụt xuống dưới **4/5 mức đỉnh** (tức sụt 1/5), đầu ra chuyển từ mức cao xuống mức thấp.
  - Khi đạn bay qua xong và tín hiệu hồi lại, đầu ra trở về mức cao.
- Kết quả là một **xung số sạch** cho mỗi lần đạn bay qua, tránh được báo sai.

## 6. Thu thập dữ liệu và thực nghiệm

Trong thực nghiệm, hệ đề xuất sai số dưới ±2 mm, chính xác gấp khoảng 2 lần hệ truyền thống (±4 mm).

### Nhận dạng tọa độ

Các xung được đưa về bo mạch thu thập dùng FPGA. Một viên đạn thường làm **vài PD liền nhau** cùng phản hồi. Ví dụ trong bài là các PD số 47–49 trên một bo. Tọa độ được tính là **tâm của nhóm PD đó**, dựa trên vị trí bắt đầu của bo, khoảng cách 4 mm giữa các PD và số PD phản hồi. Trục Y được tính theo cách tương tự.

### Bố trí thí nghiệm

Ba hệ thống được đặt nối tiếp nhau trên đường bay:

1. **Bia giấy tọa độ:** độ chính xác 0,1 mm, dùng làm **chuẩn** để so sánh.
2. **Hệ quang điện tử truyền thống:** dùng laser bán dẫn và thấu kính chuẩn trực.
3. **Hệ thống đề xuất.**

Trước khi bắn, cả ba được hiệu chuẩn chung một hệ trục. Thí nghiệm dùng súng mô phỏng bắn đạn đường kính 5,8 mm.

### Kết quả (Bảng 1)

|  | Hệ truyền thống | Hệ đề xuất |
| --- | --- | --- |
| Sai số so với bia giấy | trong khoảng ±4 mm | dưới ±2 mm |
| Độ lệch chuẩn trục X | 0,88 mm | 0,47 mm |
| Độ lệch chuẩn trục Y | 1,14 mm | 0,49 mm |

Các tác giả cho rằng hệ truyền thống sai số lớn vì **màn sáng của nó không hoàn toàn song song**, mà đó lại chính là điểm yếu hệ mới đã tránh được.

## 7. Đánh giá tổng quan

Ai cần một bia không tiếp xúc rẻ tiền thì ý tưởng này đáng tham khảo, nhưng phần kiểm chứng thực nghiệm còn mỏng.

**Điểm mạnh:**

- Ý tưởng đơn giản và rõ ràng: dùng "lọc hình học" bằng hai lớp khe thay cho thấu kính đắt tiền.
- Linh kiện phổ biến, giá rẻ, dễ lắp và dễ căn chỉnh.
- Lập luận đi theo một chuỗi chặt chẽ: mô hình hình học, rồi mô phỏng Zemax, rồi chọn ngưỡng, rồi kiểm chứng thực nghiệm.

**Điểm hạn chế, cần lưu ý khi đọc:**

- **Độ phân giải bị giới hạn bởi bước PD (4 mm).** Đạn nhỏ hơn 5,8 mm có thể không được đảm bảo sai số ±2 mm.
- **Độ nhạy không đều theo chiều sâu:** đạn bay gần phía LED khó phát hiện hơn.
- **Hai lớp khe chặn nhiều ánh sáng**, nên tỷ số tín hiệu trên nhiễu thấp hơn, dễ bị ảnh hưởng bởi ánh sáng môi trường ngoài trời.
- Bài **không nói rõ số phát bắn** trong thí nghiệm, cũng không thử với nhiều cỡ đạn hay vận tốc khác nhau. Mẫu thử nghiệm khá hạn chế.
- Bài chưa bàn tới trường hợp **nhiều viên đạn bay qua gần như cùng lúc** (bắn loạt).
- Bản PDF có một số chỗ trình bày lỗi (ví dụ "date acquisition" lẽ ra là "data acquisition", một số công thức bị lặp hoặc vỡ định dạng).

**Hướng phát triển tác giả đề xuất:** dùng PD nhỏ hơn và đặt dày hơn để tăng độ chính xác, mở rộng màn sáng cho nhiều cỡ đạn, và làm phần mềm hiển thị tọa độ theo từng khung hình.

## Nguồn

- Chu, W.; Zhang, B.; Liu, B.; Gui, Z.; Zhao, D. *An Optoelectronic Targeting System for Measuring the Distribution of Projectile Motion Based on the Subdivision of a Light Screen.* Photonics 2019, 6, 126. [doi:10.3390/photonics6040126](https://doi.org/10.3390/photonics6040126)
