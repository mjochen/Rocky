# Robot software


https://blog.robotair.io/the-complete-beginners-guide-to-using-docker-for-ros-2-deployment-2025-edition-0f259ca8b378



## ROS?

Ah, now we're getting to the heart of your project — a **sensor-rich, characterful robot with LLM-driven behavior and autonomy features** like battery monitoring and self-charging. That changes things a bit.

---

Yes — This *is* the kind of project ROS is made for

Here’s why:

### 🧩 1. **Scalable Sensor Architecture**

* ROS lets you add sensors (distance, temperature, battery, GPS, etc.) as **modular nodes**.
* You don’t have to rewrite your logic each time — you just subscribe to new topics like `/battery_level`, `/proximity`, etc.
* Debugging and monitoring via tools like `rqt_graph` and `ros2 topic echo` becomes incredibly useful.

### 🔋 2. **Battery Monitoring & Charging Logic**

* ROS doesn’t come with a “battery module” out of the box, but:

  * You can run a **battery monitor node** that reads from ADC, I2C, or smart battery interfaces.
  * It can publish data to a topic like `/battery/status`.
  * Another node (your behavior controller or decision logic) **subscribes** and decides: "Am I low? Should I say something? Should I drive to my charger?"

* Later you could add:

  * A `/charger_locator` service that uses camera + AR marker
  * A `/docking` node that runs a path planner

ROS even has some **existing packages** for autonomous docking (like from TurtleBot).

### 🤖 3. **LLM as a Sidecar**

* Your LLM logic can live **outside ROS** (on your main server) and expose a local API.
* A ROS node can then:

  * Subscribe to events (`/seen_human`, `/bored`, `/battery_low`)
  * Send a query to the LLM
  * Publish LLM responses to `/speech_output` or `/intent_action`

That keeps your AI logic and hardware decoupled — a major benefit in the long run.

---

## 🧠 Summary

| Feature                 | Without ROS               | With ROS                         |
| ----------------------- | ------------------------- | -------------------------------- |
| Add sensors over time   | Manual, error-prone       | Easy with modular nodes          |
| Battery monitoring      | DIY scripting             | Clean publisher/subscriber model |
| Autonomous docking      | Complex, custom logic     | ROS ecosystem has examples       |
| LLM integration         | Still external either way | Use ROS to trigger/consume it    |
| Scaling beyond 1 device | Hard                      | ROS 2 has multi-device support   |
| Debugging communication | Manual print/debug        | ROS tools like `rqt_graph`       |

➡️ **Verdict for Your Case**:
Since you plan to:

* Add many sensors,
* Monitor energy,
* Possibly add autonomous behaviors,
* AND want long-term growth...

👉 **Use ROS 2.** It’s worth the initial investment.

---

## 🚀 Next Steps (if you go with ROS)

1. **Start small** with:
   * A `battery_monitor` node
   * A `sensor_manager` node
   * A `llm_bridge` node (talks to your server)
2. Structure the bot’s logic using **event-based triggers** (e.g., when `/battery/status` < 20%, trigger alert).
3. Let the LLM be a *flavor* on top — the robot is a robot first, a quirky companion second.

# Robot hardware

* **Raspberry Pi 3**: Too underpowered for what you’re describing.
* **Raspberry Pi 4 (4GB or 8GB)**: Minimum viable option, *acceptable* for basic ROS 2 and sensors.
* **Better option**: Raspberry Pi 5 (or an SBC with more horsepower) if you want smooth performance and future growth.

## 🔍 Why the Pi 3 Will Struggle

| Component  | Pi 3 B+ Specs              | Problem                            |
| ---------- | -------------------------- | ---------------------------------- |
| CPU        | 4-core Cortex-A53 (1.4GHz) | Slow for multitasking, ROS 2 nodes |
| RAM        | 1 GB                       | ROS 2 + sensors + networking = 😵  |
| USB & I/O  | USB 2.0 only               | Limited sensor bandwidth           |
| OS support | 32-bit default             | ROS 2 needs 64-bit to shine        |

🔴 ROS 2 on a Pi 3 is **possible**, but you’ll quickly hit walls:

* Laggy sensor reading
* Inability to run multiple nodes reliably
* Latency in reacting to events (like low battery or human detection)
* No GPU acceleration for vision

## ✅ Raspberry Pi 4: Much Better (Especially 4GB+)

| Model        | Viable?   | Notes                                                  |
| ------------ | --------- | ------------------------------------------------------ |
| **Pi 4 2GB** | ⚠️ Barely | OK for headless setup with limited sensors             |
| **Pi 4 4GB** | ✅ Good    | Runs ROS 2 + sensors + simple camera + LLM bridge      |
| **Pi 4 8GB** | ✅ Great   | Future-proof, can buffer camera frames, run SLAM, etc. |

You can:

* Run `ros2` nodes reliably
* Stream data to/from your LLM server
* Handle camera input and basic sound I/O
* Use USB3 for faster peripherals (e.g., camera or sound card)

---

## 🚀 Raspberry Pi 5 (or Alternatives)

If budget allows, the **Pi 5 (4GB or 8GB)** is ideal:

* Much faster CPU & GPU
* Real PCIe support (coming in clutch for later vision accelerators)
* Fast I/O, great thermal design
* Good community ROS 2 support growing quickly

Or look into:

* **NVIDIA Jetson Nano / Orin Nano**: Good if you plan to add vision-heavy stuff or local inference
* **Odroid, Libre AML, LattePanda**: Often faster but more DIY in terms of community/software

---

## 🧠 Suggested Strategy

| Phase               | Recommended Board                                                      |
| ------------------- | ---------------------------------------------------------------------- |
| Early prototyping   | Raspberry Pi 4 (4GB)                                                   |
| Launch + future use | Raspberry Pi 5 or Jetson                                               |
| Budget/legacy       | Use Pi 3 only for trivial tasks (e.g., single sensor + MQTT publisher) |

> 💡 If you want the Pi only for low-level I/O (motor control, battery reading), and LLM/decision-making lives elsewhere — the Pi 4 is *just enough*. But for scaling into cool stuff like voice, vision, mapping… it’s worth getting better hardware.


## 🧱 Suggested ROS Node Architecture

| ROS Node               | Purpose                                    |
| ---------------------- | ------------------------------------------ |
| `/cmd_vel` publisher   | Sends movement commands (linear/angular)   |
| `serial_motion_bridge` | Translates `/cmd_vel` to Arduino serial    |
| `serial_eyes_bridge`   | Publishes `/mood`, `/eyes/track`           |
| `battery_monitor`      | Reads voltage from motion Arduino          |
| `navigation_core`      | Decides where to go (goal, idle, charging) |
| `llm_bridge`           | Talks to your LLM server                   |
