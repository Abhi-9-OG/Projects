# Projects
Project ideas for reference
# ESP32 Shop Order Alert & Personal Dues Tracker

Two small IoT projects built on one shared backend, using an ESP32, a FastAPI + SQLite server, and simple physical inputs and outputs (LEDs, buttons, OLED, buzzer).

| Project | Purpose |
|---|---|
| **Build A: Shop Order Alert + Smart Shelf** | Physical light and buzzer alerts for new shop orders, with optional weight-based stock sensing |
| **Build B: Personal Dues Tracker** | Press a button to log a recurring expense and see the exact amount owed on an OLED |

> **Status:** Planning and design. Backend and firmware are not yet implemented.

---

## Background

This began as a multi-tenant SaaS idea for kirana stores (QR storefront, inventory, online payments). A feasibility review showed major risks: weak customer benefit from pickup-only ordering, udhaar (credit) not fitting prepaid flows, free WhatsApp Business competition, poor unit economics at low subscription prices, and heavy onboarding and support costs.

The underlying personal need was simpler: *"give me the exact amount I owe."* These builds test that need, and the shop-alert concept, cheaply with hardware already on hand.

---

## Hardware

**Already available:** ESP32, Arduino Nano, LEDs, jumper wires, 2 breadboards.

**Small purchases (approx. cost in INR):**

| Part | Cost | Used in |
|---|---|---|
| 220Ω resistors (pack) | 20-30 | A |
| Push buttons (2 for A, 4 for B) | 10-20 | A, B |
| Active buzzer | 15-25 | A (B optional) |
| 0.96" I2C OLED (SSD1306) | 150-250 | B |
| HX711 + 5 kg load cell | 150-250 | A (optional smart shelf) |

---

## Architecture

```
ESP32 (buttons, LEDs, OLED, buzzer)
        |  HTTP over WiFi (2.4 GHz)
        v
FastAPI backend  -->  SQLite database
        ^
        |
Web dashboard (orders and dues)
```

### Database

```
orders(id, customer, items, total, status, created_at)   -- status: new | preparing | ready
entries(id, item, amount, paid, created_at)
shelf_readings(id, weight_kg, created_at)                 -- optional, smart shelf
```

### API endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/orders/summary` | Counts of orders by status |
| POST | `/orders` | Create an order |
| POST | `/orders/{id}/status` | Update order status |
| POST | `/entries` | Log an expense `{item, amount}` |
| GET | `/entries/total` | Total unpaid amount |
| POST | `/entries/settle` | Mark all entries paid |

---

## Build A: Shop Order Alert + Smart Shelf

**Behavior**

- ESP32 polls `/orders/summary` every 3 seconds.
- Red LED blinks and the buzzer beeps when a new order arrives.
- Yellow LED is on while orders are being prepared.
- Green LED is on when the latest order is ready.
- Accept button moves the oldest `new` order to `preparing`. Ready button moves it to `ready` (about 50 ms debounce).

**Pin map**

| Component | GPIO |
|---|---|
| Red / Yellow / Green LED (each with 220Ω to GND) | 25 / 26 / 27 |
| Buzzer | 14 |
| Accept / Ready buttons (to GND, `INPUT_PULLUP`) | 32 / 33 |
| HX711 data / clock (optional) | 16 / 17 |

**Smart shelf (optional):** calibrate the load cell with a known weight, post the weight every minute to `shelf_readings`, and trigger a low-stock alert (for example below 2 kg).

**Real-world test:** leave it on a shopkeeper's counter for a week and observe whether they respond to the light and buzzer.

---

## Build B: Personal Dues Tracker

**Behavior**

- Each of 4 buttons maps to a recurring item (for example Milk ₹30, Tea ₹10, Tiffin ₹80, Other).
- A press sends `{item, amount}` to `/entries`, then the OLED shows `Owed: ₹X`.
- Holding any button for 3 seconds calls `/entries/settle` and resets the total.
- If WiFi drops, presses are buffered in memory and retried.

**Pin map**

| Component | GPIO |
|---|---|
| OLED SDA / SCL | 21 / 22 |
| OLED VCC / GND | 3.3V / GND |
| Buttons (to GND, `INPUT_PULLUP`) | 32 / 33 / 25 / 26 |
| Buzzer (optional) | 14 |

**Dashboard:** total owed, per-item breakdown (`GROUP BY item`), monthly summary.

---

## Getting Started

### 1. Backend

```bash
pip install fastapi uvicorn
uvicorn main:app --host 0.0.0.0 --port 8000
```

Find your laptop's local IP (for example `192.168.1.x`) and use `http://<laptop-ip>:8000` as the server address in the firmware.

### 2. Firmware

1. Install the Arduino IDE with ESP32 board support.
2. Install libraries as needed: `Adafruit SSD1306`, `Adafruit GFX`, `HX711`.
3. Set WiFi SSID, password, and server URL in the sketch.
4. Upload to the ESP32.

---

## Practical Notes

- Avoid GPIO 34-39 for buttons (input-only, no internal pull-up).
- The ESP32 supports only 2.4 GHz WiFi.
- Use plain HTTP on a home network. Add HTTPS and an API key before exposing the server publicly.
- The ESP32 and the server must be on the same network during development.

---

## Roadmap

1. [ ] Shared backend (FastAPI + SQLite, endpoints, simple dashboard)
2. [ ] Build B: dues tracker firmware and dashboard
3. [ ] Build A: alert light, buttons, buzzer
4. [ ] Build A: smart shelf (HX711)
5. [ ] Offline buffering and retry
6. [ ] Wiring diagrams and photos in this README
7. [ ] Optional: deploy backend to a free host with HTTPS and API key

### Validation goals (for the shop concept)

- Test willingness to pay with 30-50 shopkeepers before building more.
- Set kill criteria in advance, for example fewer than 10 of 25 pilot shops still active at month three means stop or pivot.
- Prefer dense urban or semi-urban kiranas first (for example Pune and surroundings).

---

## Future Ideas

- Email/SMS parsing to auto-log dues (Gmail API or a Telegram bot)
- RFID (RC522) to identify customers or items
- Arduino Nano handling real-time I/O, linked to the ESP32 over serial
- Vivado practice: simulate a traffic-light FSM or counter in Verilog and compare with the Nano version

---

## License

Add a license of your choice (for example MIT).
