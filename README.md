## 腕带式体感鼠标

基于ATmega328p最小系统板的有线体感鼠标
<br>详见《腕带式体感鼠标.doc》
<br>[点击下载](https://github.com/lixiaoming998/Portable-somatosensory-mouse/blob/main/%E8%85%95%E5%B8%A6%E5%BC%8F%E4%BD%93%E6%84%9F%E9%BC%A0%E6%A0%87.zip)
<br>[English](https://github.com/lixiaoming998/Portable-somatosensory-mouse/blob/main/README-en.md)

## 应用

```C++
#include <Wire.h>
#include <Mouse.h>

const int ADXL345_ADDRESS = 0x53; // ADXL345 I2C地址
const int PCINT_PIN_LEFT = 8;     // PCINT8对应的引脚
const int PCINT_PIN_RIGHT = 9;    // PCINT9对应的引脚

int16_t xOffset, yOffset, zOffset; // 加速度传感器的偏移量

void setup() {
  Wire.begin();
  Mouse.begin();

  // 初始化ADXL345
  writeToADXL345(0x31, 0x09); // 设置测量范围为+-4g
  writeToADXL345(0x2D, 0x08); // 进入测量模式
  calibrateADXL345();
  
  // 配置PCINT引脚
  pinMode(PCINT_PIN_LEFT, INPUT_PULLUP);
  pinMode(PCINT_PIN_RIGHT, INPUT_PULLUP);
  PCMSK1 |= (1 << PCINT8); // 使能PCINT8
  PCMSK1 |= (1 << PCINT9); // 使能PCINT9
  PCICR |= (1 << PCIE1);   // 使能PCINT1
}

void loop() {
  // 读取加速度传感器数据
  int16_t x, y, z;
  readFromADXL345(0x32, 6, (uint8_t*)&x);
  readFromADXL345(0x34, 6, (uint8_t*)&y);
  x -= xOffset;
  y -= yOffset;

  // 映射加速度数据到鼠标移动
  int mouseX = map(x, -200, 200, -5, 5);
  int mouseY = map(y, -200, 200, -5, 5);

  // 发送鼠标移动指令
  Mouse.move(mouseX, mouseY, 0);

  // 检测微动行程开关状态
  if (digitalRead(PCINT_PIN_LEFT) == LOW) {
    Mouse.press(MOUSE_LEFT);
    delay(50);
    Mouse.release(MOUSE_LEFT);
    while (digitalRead(PCINT_PIN_LEFT) == LOW) {} // 等待按钮释放
  }

  if (digitalRead(PCINT_PIN_RIGHT) == LOW) {
    Mouse.press(MOUSE_RIGHT);
    delay(50);
    Mouse.release(MOUSE_RIGHT);
    while (digitalRead(PCINT_PIN_RIGHT) == LOW) {} // 等待按钮释放
  }
}

void writeToADXL345(byte address, byte val) {
  Wire.beginTransmission(ADXL345_ADDRESS);
  Wire.write(address);
  Wire.write(val);
  Wire.endTransmission();
}

void readFromADXL345(byte address, int num, uint8_t* buf) {
  Wire.beginTransmission(ADXL345_ADDRESS);
  Wire.write(address);
  Wire.endTransmission();
  Wire.requestFrom(ADXL345_ADDRESS, num);
  int i = 0;
  while (Wire.available()) {
    buf[i++] = Wire.read();
  }
}

void calibrateADXL345() {
  int numSamples = 50;
  int xTotal = 0, yTotal = 0, zTotal = 0;

  // 采集多次数据并计算平均值
  for (int i = 0; i < numSamples; i++) {
    int16_t x, y, z;
    readFromADXL345(0x32, 6, (uint8_t*)&x);
    readFromADXL345(0x34, 6, (uint8_t*)&y);
    xTotal += x;
    yTotal += y;
    zTotal += z;
    delay(10);
  }

  // 计算平均值并设置偏移量
  xOffset = xTotal / numSamples;
  yOffset = yTotal / numSamples;
  zOffset = zTotal / numSamples;
}
```
该代码只提供框架，未验证！

## 开源许可

[MIT](https://choosealicense.com/licenses/mit/)
