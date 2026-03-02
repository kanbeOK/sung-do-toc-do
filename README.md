# sung-do-toc-do

---

## 1. Linh kiện

Để làm một chiếc súng bắn tốc độ thực thụ, bạn cần:

* **Cảm biến Radar HB100 (10.525 GHz):** Đây là linh kiện quan trọng nhất.
* **Module Khuếch đại tín hiệu (Pre-amp):** Tín hiệu từ HB100 rất nhỏ (vài mV), cần mạch khuếch đại (thường dùng Op-amp LM358 hoặc TL082) để Arduino có thể đọc được.
* **Arduino Nano/Uno:** Xử lý tính toán.
* **Màn hình OLED 0.96 inch:** Hiển thị số vận tốc to, rõ.
* **Pin sạc 18650 (7.4V):** Cung cấp dòng điện ổn định cho Radar.

---

## 2. Công thức tính toán

Tần số Doppler ($F_d$) được tính như sau:


$$F_d = 2v \cdot \left( \frac{f_t}{c} \right) \cdot \cos(\theta)$$


Trong đó:

* $v$: Vận tốc vật thể.
* $f_t$: Tần số phát (10.525 GHz).
* $c$: Vận tốc ánh sáng.
* $\theta$: Góc giữa súng và hướng di chuyển (thường để $0^\circ$ để đạt độ chính xác cao nhất).

Rút gọn cho cảm biến HB100, ta có công thức thực tế:
**Vận tốc (km/h) = $F_d$ (Hz) / 19.49**

---

## 3. Sơ đồ đấu nối 

1. **HB100 VCC/GND** nối vào nguồn 5V sạch.
2. **HB100 IF (Output)** nối vào đầu vào mạch khuếch đại Op-amp.
3. **Đầu ra Op-amp** nối vào chân **Digital Pin 5** của Arduino 

---

## 4. Code 



```cpp
#include <FreqMeasure.h>

double sum = 0;
int count = 0;

void setup() {
  Serial.begin(57600);
  FreqMeasure.begin(); // Bắt đầu đo tần số trên chân D5 (Uno) hoặc D4 (Nano)
}

void loop() {
  if (FreqMeasure.available()) {
    // Đo trung bình 30 mẫu để ổn định kết quả
    sum = sum + FreqMeasure.read();
    count = count + 1;
    
    if (count > 30) {
      float frequency = FreqMeasure.countToFrequency(sum / count);
      float speedKmH = frequency / 19.49; // Công thức cho HB100
      
      if (speedKmH > 1.0) { // Loại bỏ nhiễu khi đứng yên
        Serial.print("Toc do: ");
        Serial.print(speedKmH);
        Serial.println(" km/h");
      }
      
      sum = 0;
      count = 0;
    }
  }
}

```

