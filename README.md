# FB_ProductionRateMonitor (Omron Sysmac / Structured Text)

A lightweight, reusable **Function Block** for Omron NJ/NX series PLCs (Sysmac Studio) that monitors real-time production performance — part count, cycle time, and production rate (parts/minute) — using a simple product presence sensor input.

---

## 📌 Overview

`FB_ProductionRateMonitor` tracks parts produced on a line/station by detecting rising edges on a product presence sensor. It automatically:

- Counts produced parts up to a target quantity
- Measures total elapsed production time
- Calculates **average cycle time per part**
- Calculates **production rate in Parts Per Minute (PPM)**
- Supports a manual reset to clear all counters and start fresh

This FB is ideal for **OEE dashboards, line monitoring HMIs, and throughput tracking** on Omron machines.

---

## ⚙️ How It Works

1. **Edge Detection** – A rising-edge trigger (`fb_ProductDetectionEdge`) detects each new part passing the sensor (`i_ProductPresenceSensor`).
2. **Part Counting** – Each detected part increments `ProducedPartCount`, until the target quantity (`i_TargetProductionQuantity`) is reached.
3. **Timer** – `fb_ProductionElapsedTimer` (TON) runs while measurement is active, tracking elapsed time.
4. **Rate Calculation**:
   - Total time is converted to nanoseconds (`TimeToNanoSec`)
   - Average cycle time = Total time ÷ Parts produced
   - Production rate (PPM) = `(ProducedPartCount × 60,000,000,000.0) / TotalProductionTimeNs`
5. **Auto-Stop** – Measurement stops automatically once the target quantity is reached.
6. **Reset** – Setting `i_ResetProductionMeasurement` clears all counters, timers, and rate values.

---

## 🔌 Inputs

| Variable                        | Type | Description                                      |
|----------------------------------|------|---------------------------------------------------|
| `i_ProductPresenceSensor`        | BOOL | Sensor input signaling a part is present           |
| `i_TargetProductionQuantity`     | INT  | Target number of parts to produce in this cycle    |
| `i_ResetProductionMeasurement`   | BOOL | Resets all counters, timers, and calculated values |

## 📤 Outputs

| Variable          | Type  | Description                                  |
|--------------------|-------|-----------------------------------------------|
| `o_PartPerMinut`   | LREAL | Real-time production rate (parts per minute)   |

## 🧮 Internal Variables (key ones)

| Variable                       | Type | Description                              |
|----------------------------------|------|-------------------------------------------|
| `ProducedPartCount`             | INT/DINT | Running count of parts produced        |
| `x_CycleTimeMeasurementActive`  | BOOL | Flag indicating measurement is running   |
| `t_ProductionElapsedTime`       | TIME | Live elapsed time from the timer         |
| `t_TotalProductionTime`         | TIME | Captured total production time           |
| `li_TotalProductionTimeNs`      | LINT | Production time converted to nanoseconds |
| `li_AverageCycleTimeNs`         | LINT | Average cycle time per part (ns)         |
| `t_AverageCycleTimePerPart`     | TIME | Average cycle time per part (TIME format)|

---

## 🛠️ Requirements

- **Omron NJ/NX Series PLC**
- **Sysmac Studio** (Structured Text support)
- Standard library function blocks:
  - `R_TRIG` (rising edge, used as `fb_ProductDetectionEdge`)
  - `TON` (on-delay timer, used as `fb_ProductionElapsedTimer`)
- Built-in conversion functions: `TimeToNanoSec`, `NanoSecToTime`, `INT_TO_LREAL`, `LINT_TO_LREAL`

---

## 🚀 Usage

1. Add `FB_ProductionRateMonitor` to your Sysmac Studio project.
2. Create an instance of the FB in your main program.
3. Wire up the sensor input and target quantity:

```st
fbProdRateMonitor(
    i_ProductPresenceSensor      := SensorInput,
    i_TargetProductionQuantity   := 500,
    i_ResetProductionMeasurement := ResetButton
);

CurrentPPM := fbProdRateMonitor.o_PartPerMinut;
```

4. Read `o_PartPerMinut` for live rate display on your HMI/SCADA.
5. Pulse `i_ResetProductionMeasurement` to start a new production run.

---

## 📈 Example Use Case

On a packaging line producing 500 units per batch, this FB gives:
- Live parts/minute readout for operators
- Average cycle time per part for cycle-time analysis
- Automatic stop once the batch target is met — ready for the next run after reset

---

## 📄 License

Free to use and modify for industrial automation projects. Attribution appreciated but not required.

---

## 🙋‍♂️ Author

Created by **Animesh Kumar** — feel free to connect on LinkedIn for feedback, improvements, or automation project discussions.
