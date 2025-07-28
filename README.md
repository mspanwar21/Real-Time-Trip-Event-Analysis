# 🚕 Real-Time Trip Event Analysis – CST8917 Lab 4

## 📘 Overview

This project demonstrates a real-time event-driven system for monitoring taxi trip data using Azure Event Hub, Azure Functions, and Logic Apps. The system automatically analyzes each trip, flags unusual patterns, and sends rich Adaptive Card alerts to Microsoft Teams for dispatcher awareness.

---

## 📊 Architecture

![Architecture Diagram](flowchart.png)

**Components:**
- **Event Hub:** Ingests simulated taxi trip events.
- **Azure Function:** Analyzes each trip for patterns such as group rides, cash payments, and suspicious short trips.
- **Logic App:** Routes function output and posts Adaptive Cards to Microsoft Teams.
- **Microsoft Teams:** Receives alerts via webhook for easy dispatcher visibility.

---

## 🔍 Azure Function Logic

```python
import azure.functions as func
import logging
import json

app = func.FunctionApp(http_auth_level=func.AuthLevel.ANONYMOUS)

@app.route(route="")
def analyze_trip(req: func.HttpRequest) -> func.HttpResponse:
    try:
        input_data = req.get_json()
        trips = input_data if isinstance(input_data, list) else [input_data]

        results = []
        for record in trips:
            trip = record.get("ContentData", {})
            vendor = trip.get("vendorID")
            distance = float(trip.get("tripDistance", 0))
            passenger_count = int(trip.get("passengerCount", 0))
            payment = str(trip.get("paymentType"))
            insights = []

            if distance > 10:
                insights.append("LongTrip")
            if passenger_count > 4:
                insights.append("GroupRide")
            if payment == "2":
                insights.append("CashPayment")
            if payment == "2" and distance < 1:
                insights.append("SuspiciousVendorActivity")

            results.append({
                "vendorID": vendor,
                "tripDistance": distance,
                "passengerCount": passenger_count,
                "paymentType": payment,
                "insights": insights,
                "isInteresting": bool(insights),
                "summary": f"{len(insights)} flags: {', '.join(insights)}" if insights else "Trip normal"
            })

        return func.HttpResponse(
            body=json.dumps(results),
            status_code=200,
            mimetype="application/json"
        )

    except Exception as e:
        logging.error(f"Error processing trip data: {e}")
        return func.HttpResponse(f"Error: {str(e)}", status_code=400)
```

---

## ⚙️ Logic App Workflow

- **Trigger**: Event Hub (batch mode)
- **HTTP Request**: Call Azure Function with batch body
- **For Each**: Iterate over Function results
  - **Condition**: `isInteresting == true`
    - If contains `SuspiciousVendorActivity`: Post Suspicious Card
    - Else: Post Interesting Card
  - **Else**: Post Normal Trip Card

🔁 See `logic-app-trip-monitoring.json`

---

## 🧪 Sample Input

```json
{
  "ContentData": {
    "vendorID": "V003",
    "tripDistance": "0.4",
    "passengerCount": "1",
    "paymentType": "2"
  }
}
```

## ✅ Sample Output

```json
[
  {
    "vendorID": "V003",
    "tripDistance": 0.4,
    "passengerCount": 1,
    "paymentType": "2",
    "insights": ["CashPayment", "SuspiciousVendorActivity"],
    "isInteresting": true,
    "summary": "2 flags: CashPayment, SuspiciousVendorActivity"
  }
]
```

---

## 💬 Adaptive Cards

Adaptive Cards used in Microsoft Teams:
- **Not Interesting Trip**
- **Interesting Trip**
- **Suspicious Vendor Activity**

Cards are posted using the `HTTP` connector and a pre-configured Incoming Webhook in Teams.

---

## 📽️ Demo Video

[![YouTube Demo](https://img.shields.io/badge/Watch-Demo%20Video-red?logo=youtube)](https://youtu.be/YOUR_VIDEO_LINK)

---

## 🚀 Improvements & Learnings

### Potential Enhancements
- Store flagged trips in Azure Cosmos DB for future analysis
- Add Power BI dashboard to visualize trip patterns
- Use Logic App retry policies for reliability
- Integrate Azure Cognitive Services for NLP insights on feedback

### Lessons Learned
- How to use Azure Event Hub with Logic Apps
- How to analyze data in real time using Azure Functions
- How to send Adaptive Cards to Teams using Logic Apps

---

## 📁 Project Structure

```
.
├── logic-app-trip-monitoring.json
├── architecture.png
├── README.md
```

---

## 👨‍💻 Author

**Mohit Singh Panwar**  
CST8917 – DevOps Security and Compliance  
Algonquin College  

