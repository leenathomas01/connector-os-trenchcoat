# Recipe: PROMETHEUS-1 (Apple Shortcuts)

> **Platform:** iOS / watchOS  
> **Difficulty:** Beginner (No coding, just drag-and-drop)  
> **Time:** ~15 Minutes  
> **Prerequisites:** iPhone, Apple Watch (for HRV), Smart Lights (optional)

---

## 1. Concept
This recipe builds a **closed-loop stress regulator** using tools you already own.
It does not use "AI" to talk to you. It uses simple math to check your biological energy (Heart Rate Variability) and modulates your environment to match your state.

**Why HRV?**
Heart Rate Variability is the cleanest non-invasive proxy for cognitive load, autonomic balance, and recovery. When you are stressed, HRV drops.

**The Math:** We calculate **Deviation**: `(Baseline - Current) / Baseline`.
- If your HRV drops **15%**, we nudge (Yellow State).
- If your HRV drops **30%**, we intervene (Red State).

---

## 2. Preparation (The "Actuators")

Before building the Shortcut, set up the outputs you want to trigger.

**Option A: Smart Lights (HomeKit)**
1.  Create a Scene named **"Zen Mode"**: Warm color (Amber/Orange), 40% brightness.
2.  Create a Scene named **"Focus Mode"**: Cool color (Blue/White), 80% brightness.

**Option B: No Smart Lights? (Fallbacks)**
- You can use **"Set Brightness"** (dim the screen).
- You can use **"Play Sound"** (Brown Noise).
- You can use **"Vibrate Device"**.

**Focus Modes (iOS Settings):**
- Ensure you have a **"Work"** focus mode set up (ideal for running this automation).
- Ensure you have a **"Driving"** focus mode set up (for safety interlocks).

---

## 3. The Construction (Step-by-Step)

Open the **Shortcuts App** and create a new Shortcut named `PROMETHEUS-1`.

### Part A: The Safety Interlocks
*We never intervene if you are driving or working out.*

1.  **Action:** `Get Current Focus`
2.  **Action:** `If` [Current Focus] `Name` contains "Driving"
    * **Action:** `Stop This Shortcut`
3.  **End If**
4.  **Action:** `Get Motion Activity` (optional)
    * *Logic:* If sedentary or still, proceed. (You can skip this for v1).

### Part B: Establishing Baseline (The Alphabet)
*We need to know what "Normal" is for YOU.*

1.  **Action:** `Find Health Samples`
    * **Type:** Heart Rate Variability
    * **Start Date:** is in the last `7` days
    * **Sort by:** Date (Latest First)
    * **Limit:** (Leave blank or set high, e.g., 50)
2.  **Action:** `Calculate Statistics`
    * **Operation:** Average
    * **Input:** [Health Samples]
3.  **Action:** `Set Variable`
    * **Name:** `Baseline`
    * **Value:** [Average]

### Part C: Reading Current State (The Input)
*What is happening right now?*

1.  **Action:** `Find Health Samples`
    * **Type:** Heart Rate Variability
    * **Start Date:** is in the last `1` hours
    * **Limit:** (Leave blank)
2.  **Action (CRITICAL FAIL-SAFE):** `If` [Health Samples] `has no value`
    * **Action:** `Stop This Shortcut`
    * *Why: Prevents false alarms if the watch hasn't taken a reading recently.*
3.  **End If**
4.  **Action:** `Calculate Statistics`
    * **Operation:** Average
    * **Input:** [Health Samples]
5.  **Action:** `Set Variable`
    * **Name:** `Current`
    * **Value:** [Average]

### Part D: The Connector Logic (The "Grid")
*Calculate the stress deviation.*

1.  **Action:** `Calculate`
    * **Input:** `Baseline` - `Current`
2.  **Action:** `Calculate`
    * **Input:** [Calculation Result] / `Baseline`
3.  **Action:** `Set Variable`
    * **Name:** `Deviation`

### Part E: Actuation (The Output)
*Determine the State Band and act.*

1.  **Action:** `If` [Deviation] is greater than `0.30` (Red State)
    * **Action:** `Show Notification`
        * **Title:** "High Cognitive Load"
        * **Body:** "Activate Zen Mode?"
        * **Options:** Show "Run" button (Interactive).
    * *Note: We ask permission first (Guardian, not Overlord).*
    * **Action:** `Control Home` -> Set "Zen Mode" OR `Play Sound` (Brown Noise).
    * **Action:** `Vibrate Device` (Haptic Ticker).
2.  **Otherwise**
    3.  **Action:** `If` [Deviation] is greater than `0.15` (Yellow State)
        * **Action:** `Control Home` -> Set "Zen Mode" (but maybe brighter).
        * *Note: A subtle nudge without asking.*
    4.  **End If**
3.  **End If**

---

## 4. Automation (The Loop)

A Shortcut doesn't run itself. You need an Automation trigger.

1.  Go to **Shortcuts > Automation**.
2.  **Trigger:** When **"Work" Focus** turns ON.
    * *Why: This naturally limits the system to monitoring you only when you are working.*
3.  **Trigger (Optional):** Time of Day (9am, 11am, 1pm, 3pm) if you don't use Focus Modes.
4.  **Action:** Run Shortcut `PROMETHEUS-1`.
5.  **Options:** Turn OFF "Ask Before Running" (The logic inside handles the permission).

---

## 5. Troubleshooting / Tuning

- **"It triggers too often!"** -> Your HRV fluctuates a lot. Increase the Red Threshold to `0.40`.
- **"It never triggers!"** -> Decrease the Threshold to `0.20`.
- **"It crashes!"** -> Ensure Part C (The Fail-Safe) is correctly implemented.
- **Pro Tip (v1.1):** If you are an athlete with massive HRV swings, future versions will use "Z-Score" (Standard Deviation) instead of simple percentage deviation.

---

## 6. Compatibility Note

**Android Users:**
This specific recipe relies on Apple HealthKit and Shortcuts. However, the logic is universal. You can replicate this on Android using:
- **Tasker** (Automation)
- **Google Fit / Health Connect** (HRV Data)
- **Hue / SmartThings** (Actuators)

---

> **Philosophy Check:**
> The goal isn't to force you to relax.
> The goal is to reduce the latency between "My body is stressed" and "I know I am stressed."
> The lights turning amber is just a signal. You are the pilot.
