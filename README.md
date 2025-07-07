# Real-Time-DataProcessing

Dynamic Pricing for Urban Parking Lots

**Capstone Project of Summer Analytics 2025**
**Hosted by: Consulting & Analytics Club × Pathway**

---

## Project Overview

Urban parking spaces are a limited and highly demanded resource. Static pricing models are inefficient, often leading to either underutilized lots or overcrowded spaces with long queues. This project tackles this problem by creating an intelligent, data-driven dynamic pricing engine.

The system simulates a real-time data stream from urban parking lots and uses this data to adjust prices dynamically, aiming to improve utilization and respond effectively to real-world conditions like demand, traffic, and special events.

---

## Project Objective

The primary goal is to build a dynamic pricing model that updates prices in real-time for urban parking lots. The model must be:
*   **Data-Driven:** Prices are based on historical patterns, queue length, traffic, special events, and vehicle types.
*   **Smooth and Explainable:** Price variations are logical and bounded, not erratic.
*   **Built from Scratch:** All pricing logic is implemented using only Python, Pandas, Numpy, and the Pathway library for real-time processing.

---

## Technology Stack

*   **Data Manipulation:** Python, Pandas, NumPy
*   **Real-Time Stream Processing:** Pathway
*   **Interactive Visualization:** Bokeh, Panel
*   **Environment:** Google Colab

---

## Methodology and Model Implementation

The core of the project is a Pathway streaming pipeline that ingests parking data, applies pricing logic, and outputs the results for visualization. The data is aggregated on a **daily tumbling window**, meaning a single price is calculated for each day, representing the average conditions across all lots.

### 1. Data Preparation and Feature Engineering

The provided dataset contained only `Timestamp`, `Occupancy`, and `Capacity`. To build the sophisticated models required by the project statement, the following features were synthetically generated:

*   **QueueLength:** Simulated to be higher when a lot's occupancy is high.
*   **TrafficCongestion:** Simulated to be higher during typical morning and evening rush hours.
*   **IsSpecialDay:** A flag set to `1` for weekends (Saturday/Sunday) to represent higher demand periods.
*   **VehicleTypeWeight:** A numeric weight assigned to incoming vehicles (`car`: 1.0, `truck`: 1.5, `bike`: 0.7) to model their different impact on pricing.

This enriched dataset was then streamed into the Pathway pipeline.

### 2. Model 1: Baseline Linear Model

This model serves as a simple reference point. It establishes a direct, linear relationship between the average daily occupancy and the price.

**Logic:**
`Price_day = BasePrice + α * (AverageDailyOccupancy / AverageCapacity)`

**Justification:**
*   **Simplicity:** It provides a clear, understandable baseline.
*   **Intuition:** As the average occupancy of the lots increases over a day, the price increases proportionally.
*   **Parameters:**
    *   `BasePrice = $10`
    *   `α (Alpha) = 5.0`: A sensitivity factor determining how strongly occupancy affects the price.

### 3. Model 2: Demand-Based Price Function

This is a more advanced and realistic model that calculates a composite "demand score" based on multiple real-time factors.

**Logic:**
`Price_day = BasePrice ⋅ (1 + λ ⋅ NormalizedDemand)`

**The Demand Function Explained:**

The price is driven by a `demand_score` calculated as follows:

`demand_score = (w_occ * avg_occ_rate + w_queue * avg_queue + w_traffic * avg_traffic + w_special * is_special) * avg_vehicle_weight`

*   **Components and Weights (`w`):**
    *   `avg_occ_rate`: The average occupancy rate for the day. (Weight: 2.0)
    *   `avg_queue`: The average number of cars waiting. (Weight: 1.5)
    *   `avg_traffic`: The average traffic congestion level. (Weight: 1.0)
    *   `is_special`: A flag for weekends/events. (Weight: 2.5)
    *   `avg_vehicle_weight`: A multiplier for vehicle type.
*   **Normalization:** The raw `demand_score` is passed through a **logistic (sigmoid) function**. This is a crucial step to map the potentially unbounded score into a smooth, predictable `[0, 1]` range. This prevents extreme, erratic price jumps and ensures the demand factor is well-behaved.
*   **Price Calculation:** The `NormalizedDemand` is multiplied by a sensitivity factor `λ (Lambda)` and added to the `BasePrice`, creating a price that scales intelligently with overall demand.

### A Note on Model 3 (Optional Competitive Model)

The problem statement suggested an optional third model that incorporates competitor pricing. This model was **not implemented** in the final code because the chosen aggregation strategy (`windowby(day)`) is fundamentally incompatible with it.

*   **Reason:** To compare competitor prices, the system must calculate and track the price for **each individual lot**. The current code aggregates data from all lots into a single daily price, losing the per-lot information needed for competitive analysis.
*   **Path to Implementation:** To build Model 3, one would need to change the core logic from `windowby(day)` to `groupby(LotID)` to maintain a separate state and price for each parking lot.

---

## Price Dynamics and Assumptions

### How Price Changes with Demand

*   **Model 1:** Price increases linearly with the average daily occupancy rate.
*   **Model 2:** Price increases non-linearly based on a combination of factors. A weekend (`IsSpecialDay=1`) with high traffic and long queues will result in a significantly higher price than a quiet weekday, even if the occupancy rate is similar. The price change is smooth due to the logistic normalization.

### Key Assumptions

1.  **Synthetic Data:** Key features like `QueueLength`, `TrafficCongestion`, and `VehicleType` were not in the original dataset and were generated based on logical assumptions (e.g., traffic is higher at 9 AM). In a production system, this data would come from real-time APIs or sensors.
2.  **Aggregated Pricing:** The final implementation calculates a single, system-wide price per day, not a per-lot price. This was done to adhere to the structure of the provided base code.
3.  **Fixed Model Weights:** The weights used in the Model 2 demand function were chosen based on domain expertise for demonstration purposes. In a real-world scenario, these would be fine-tuned or learned using machine learning techniques on historical data to optimize for revenue or utilization.
4.  **Bounded Prices:** All prices are capped between `0.5x` and `2.0x` the base price (`$5` to `$20`) to ensure they remain within a reasonable range.

---

## Conclusion

This project successfully demonstrates the design and implementation of a real-time dynamic pricing pipeline using Pathway. By progressing from a simple linear model to a sophisticated multi-factor demand model, it showcases how real-time data can be leveraged to create intelligent and responsive pricing strategies that are superior to traditional static methods.
