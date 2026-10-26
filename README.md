# DS18B20 Reads -127 °C or 85 °C — Diagnostic Code & Wiring Checklist

Diagnostic test sketch, wiring table, and error decoder for fixing `-127 °C` (`DEVICE_DISCONNECTED_C`), `85 °C`, and `"Sensors found: 0"` on Arduino Uno/Nano and ESP32.

---

### 1. Error Decoder: `-127 °C` vs `85 °C`
* **`-127 °C` (`DEVICE_DISCONNECTED_C`):** The sensor didn't answer at all, or the data failed the CRC check -> check wiring, the **4.7 kΩ pull-up resistor**, and the GPIO pin.
* **`85 °C` (Power-on Reset Value):** The sensor answered, but never completed a temperature conversion -> make sure you call `sensors.requestTemperatures()` before reading, and use **3-wire normal power** instead of parasite power.
* **Note on `DallasTemperature` 4.0.6+:** Newer versions convert an unread `85 °C` power-on register value into `-127 °C`, making a missing `requestTemperatures()` call look like a wiring error! Always check `sensors.getDeviceCount()` first.

---

### 2. Diagnostic Test Sketch (ESP32 & Arduino Uno)
Requires **OneWire** (`v2.3.8+` for ESP32 Arduino core 3.x compatibility) and **DallasTemperature** libraries:

```cpp
#include <OneWire.h>
#include <DallasTemperature.h>

#define ONE_WIRE_BUS 4   // ESP32: GPIO 4  |  Arduino Uno: use 2

OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

void setup() {
  Serial.begin(115200);
  sensors.begin();
  Serial.print("Sensors found: ");
  Serial.println(sensors.getDeviceCount());   // 0 = wiring / pull-up / pin
  Serial.print("Parasite mode: ");
  Serial.println(sensors.isParasitePowerMode() ? "yes" : "no");
}

void loop() {
  sensors.requestTemperatures();               // start a conversion (waits by default)
  float t = sensors.getTempCByIndex(0);
  if (t == DEVICE_DISCONNECTED_C) {            // -127
    Serial.println("Error: no valid reading");
  } else {
    Serial.print("Temperature: ");
    Serial.print(t);
    Serial.println(" C");
  }
  delay(1000);
}
```

---

### 3. Wiring & Pull-Up Resistor Checklist

| Sensor Pin (TO-92 Flat Face Toward You, Left -> Right) | Waterproof Probe Wire (Typical) | **ESP32 DevKit** | **Arduino Uno / Nano** |
| :--- | :--- | :--- | :--- |
| **Pin 1: GND** (Left) | Black | `GND` | `GND` |
| **Pin 2: DQ / DATA** (Middle) | Yellow or White | `GPIO 4` *(avoid input-only 34–39)* | `D2` |
| **Pin 3: VDD** (Right) | Red | `3V3` *(never 5V)* | `5V` |
| **Pull-Up Resistor** | **4.7 kΩ** (or **2.2 kΩ** for long cables) | Between `DQ (GPIO 4)` and `3V3` | Between `DQ (D2)` and `5V` |

* **Warning:** If the TO-92 package gets hot immediately, `GND` and `VDD` are reversed—unplug power right away!
