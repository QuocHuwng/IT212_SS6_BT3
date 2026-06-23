
# BÀI 2: THỰC HÀNH XÂY DỰNG THUẬT TOÁN KHUYẾN MÃI PHỨC TẠP (CHAIN-OF-THOUGHT - CoT)

## 1. Tầm quan trọng của thứ tự ưu tiên tính toán trong nghiệp vụ kế toán/tài chính

Trong các hệ thống thương mại điện tử, kế toán và tài chính, thứ tự thực hiện các phép tính là yếu tố cực kỳ quan trọng vì mỗi bước tính toán sẽ ảnh hưởng trực tiếp đến kết quả của các bước tiếp theo.

Nếu áp dụng sai thứ tự:

* Tính sai số tiền khách hàng phải thanh toán.
* Tính sai thuế VAT.
* Tính sai mức giảm giá và chiết khấu.
* Gây thất thoát doanh thu cho doanh nghiệp.
* Vi phạm quy định về kế toán và thuế.

Ví dụ:

* Nếu VAT được tính trước khi áp dụng các khoản giảm giá thì khách hàng sẽ phải trả thuế trên phần tiền đáng lẽ được giảm.
* Nếu Loyalty Discount được tính trước Coupon Discount thì tổng tiền cuối cùng sẽ khác với quy định nghiệp vụ.

Vì vậy trước khi lập trình cần xác định rõ:

1. Thứ tự xử lý nghiệp vụ.
2. Công thức tính cho từng bước.
3. Kiểm thử bằng tay (Dry-run).
4. Sau đó mới triển khai thành mã nguồn.

Đây chính là lý do kỹ thuật Chain-of-Thought (CoT) đặc biệt phù hợp với các bài toán tài chính có nhiều quy tắc phụ thuộc lẫn nhau.

---

# 2. Nội dung Prompt CoT được thiết kế

```text
Bạn là một Senior Java Developer đồng thời là chuyên gia phân tích nghiệp vụ E-Commerce.

Nhiệm vụ của bạn là thiết kế thuật toán tính tổng tiền đơn hàng cho hệ thống SpeedyCart.

Đừng viết code ngay.

Hãy thực hiện chính xác theo quy trình sau:

BƯỚC 1 - PHÂN TÍCH NGHIỆP VỤ

Xác định thứ tự thực hiện đúng của các bước:

1. Tính tổng tiền gốc.
2. Áp dụng Product Discount.
3. Áp dụng Coupon Discount.
4. Áp dụng Loyalty Discount.
5. Tính VAT.
6. Tính Shipping Fee.
7. Tính tổng tiền cuối cùng.

Giải thích vì sao thứ tự đó là đúng.

BƯỚC 2 - LIỆT KÊ CÔNG THỨC

Viết công thức chi tiết cho từng bước.

Ví dụ:

Tổng tiền gốc
= Σ(Giá sản phẩm × Số lượng)

VAT
= Tổng tiền sau giảm giá × 10%

...

BƯỚC 3 - DRY-RUN BẰNG TAY

Thực hiện tính toán chi tiết với dữ liệu:

- Sản phẩm A:
  + Giá gốc: 400.000 VND
  + Số lượng: 2
  + Giảm giá trực tiếp: 10%

- Sản phẩm B:
  + Giá gốc: 300.000 VND
  + Số lượng: 1
  + Không giảm giá

- Coupon:
  + Giảm 50.000 VND
  + Điều kiện đơn tối thiểu 500.000 VND

- Khách hàng:
  + Hạng Gold
  + Giảm 5%

- Khoảng cách giao hàng:
  + 12 km

Hiển thị toàn bộ phép tính bằng văn bản.

BƯỚC 4 - KIỂM TRA KẾT QUẢ

Tóm tắt:

- Tổng tiền gốc
- Product Discount
- Coupon Discount
- Loyalty Discount
- VAT
- Shipping Fee
- Tổng thanh toán cuối cùng

BƯỚC 5 - SINH MÃ NGUỒN

Dựa trên toàn bộ phân tích ở trên:

Sinh class Java OrderCalculator.

Yêu cầu:

- Java 17
- BigDecimal
- Clean Code
- Không sử dụng double

Phương thức:

public BigDecimal calculateTotal(Order order)

Có comment giải thích từng bước xử lý.
```

---

# 3. Minh chứng chạy thực tế (Phản hồi của AI)

## Bước 1: Phân tích logic nghiệp vụ

### Thứ tự xử lý đúng

1. Tính tổng tiền gốc.
2. Áp dụng Product Discount.
3. Áp dụng Coupon Discount.
4. Áp dụng Loyalty Discount.
5. Tính Shipping Fee.
6. Tính VAT.
7. Tính tổng thanh toán cuối cùng.

### Giải thích

* Product Discount phải được áp dụng đầu tiên vì đây là giảm giá trực tiếp trên từng sản phẩm.
* Coupon chỉ được áp dụng sau khi đã xác định giá trị đơn hàng thực tế.
* Loyalty Discount áp dụng trên số tiền còn lại sau Coupon.
* Shipping được tính dựa trên tổng tiền sau giảm giá và trước thuế.
* VAT phải được tính sau khi đã trừ toàn bộ các khoản giảm giá theo nguyên tắc kế toán.

---

## Bước 2: Công thức tính toán

### Tổng tiền gốc

```text
Tổng tiền gốc
= Σ(Giá sản phẩm × Số lượng)
```

### Product Discount

```text
Tiền sau Product Discount
= Tổng tiền gốc - Tổng giảm giá sản phẩm
```

### Coupon Discount

```text
Tiền sau Coupon
= Tiền sau Product Discount - Coupon
```

### Loyalty Discount

```text
Tiền sau Loyalty
= Tiền sau Coupon - Loyalty Discount
```

### Shipping Fee

```text
Nếu tổng tiền sau Loyalty > 1.000.000
=> Miễn phí vận chuyển

Ngược lại:

Dưới 10 km:
30.000

Mỗi km vượt quá:
+5.000/km
```

### VAT

```text
VAT
= Tiền sau Loyalty × 10%
```

### Tổng thanh toán

```text
Final Total
= Tiền sau Loyalty
+ Shipping Fee
+ VAT
```

---

## Bước 3: Dry-run chi tiết

### Tính tổng tiền gốc

Sản phẩm A:

```text
400.000 × 2
= 800.000
```

Sản phẩm B:

```text
300.000 × 1
= 300.000
```

Tổng tiền gốc:

```text
800.000 + 300.000

= 1.100.000
```

---

### Product Discount

Giảm giá sản phẩm A:

```text
800.000 × 10%

= 80.000
```

Tiền sau Product Discount:

```text
1.100.000 - 80.000

= 1.020.000
```

---

### Coupon Discount

Điều kiện áp dụng:

```text
1.020.000 > 500.000

=> Hợp lệ
```

Áp dụng Coupon:

```text
1.020.000 - 50.000

= 970.000
```

---

### Loyalty Discount

Khách hàng Gold:

```text
970.000 × 5%

= 48.500
```

Tiền sau Loyalty:

```text
970.000 - 48.500

= 921.500
```

---

### Shipping Fee

Khoảng cách giao hàng:

```text
12 km
```

Tính phí:

```text
30.000

+ (12 - 10) × 5.000

= 40.000
```

Không được miễn phí vận chuyển vì:

```text
921.500 < 1.000.000
```

---

### VAT

```text
921.500 × 10%

= 92.150
```

---

### Tổng thanh toán cuối cùng

```text
921.500
+ 40.000
+ 92.150

= 1.053.650 VND
```

---

## Bước 4: Kết quả tổng hợp

```text
Tổng tiền gốc:          1.100.000
Product Discount:         80.000
Coupon Discount:          50.000
Loyalty Discount:         48.500
Shipping Fee:             40.000
VAT:                      92.150
---------------------------------
TOTAL:                 1.053.650 VND
```

---

## Bước 5: Class Java OrderCalculator

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

public class OrderCalculator {

    private static final BigDecimal VAT_RATE =
            new BigDecimal("0.10");

    private static final BigDecimal GOLD_RATE =
            new BigDecimal("0.05");

    private static final BigDecimal PLATINUM_RATE =
            new BigDecimal("0.10");

    public BigDecimal calculateTotal(Order order) {

        BigDecimal subtotal =
                order.getOriginalAmount();

        // Product Discount
        subtotal =
                subtotal.subtract(
                        order.getProductDiscount());

        // Coupon Discount
        subtotal =
                subtotal.subtract(
                        order.getCouponDiscount());

        // Loyalty Discount
        if ("GOLD".equals(order.getMemberType())) {

            subtotal =
                    subtotal.subtract(
                            subtotal.multiply(GOLD_RATE));
        }

        if ("PLATINUM".equals(order.getMemberType())) {

            subtotal =
                    subtotal.subtract(
                            subtotal.multiply(PLATINUM_RATE));
        }

        // Shipping Fee
        BigDecimal shipping =
                calculateShipping(
                        subtotal,
                        order.getDistanceKm());

        // VAT
        BigDecimal vat =
                subtotal.multiply(VAT_RATE);

        return subtotal
                .add(shipping)
                .add(vat)
                .setScale(0, RoundingMode.HALF_UP);
    }

    private BigDecimal calculateShipping(
            BigDecimal subtotal,
            int distanceKm) {

        if (subtotal.compareTo(
                new BigDecimal("1000000")) > 0) {

            return BigDecimal.ZERO;
        }

        BigDecimal shipping =
                new BigDecimal("30000");

        if (distanceKm > 10) {

            shipping =
                    shipping.add(
                            BigDecimal.valueOf(
                                    (distanceKm - 10) * 5000L));
        }

        return shipping;
    }
}
```

## Kết luận

Kỹ thuật Chain-of-Thought giúp AI phân tích bài toán theo từng bước, xác định đúng thứ tự ưu tiên tính toán, kiểm chứng bằng Dry-run và sau đó mới sinh mã nguồn. Đây là phương pháp đặc biệt hiệu quả đối với các nghiệp vụ kế toán, tài chính và thương mại điện tử có nhiều quy tắc phụ thuộc lẫn nhau.
