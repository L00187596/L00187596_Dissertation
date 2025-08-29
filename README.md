# Hybrid AutoScaler with Forecasting and Reinforcement Learning

## Overview
This dissertation work demonstrates that integrating Prophet and LSTM forecasting with PPO reinforcement learning can significantly mitigate cold starts and resource inefficiencies in serverless systems. The successful deployment on Azure Functions validates the approach as a cloud-ready solution, showing that prediction-driven autoscaling achieves both performance and cost efficiency without overprovisioning.


---

## Features
- Data Preprocessing: Downloads and cleans Azure Blob logs into continuous invocation time series.
- Forecasting: Prophet, LSTM, and a Hybrid model for short- and long-term predictions.
- Evaluation: Reports MAE & RMSE with full and next-day forecasts.
- Reinforcement Learning: PPO agent learns autoscaling strategies in a custom Gymnasium environment.
- Deployment: Packaged into a Docker image and deployed on Azure Functions for real-time autoscaling decisions.

---

## Project Structure

### Preprocessing
- **`preprocess.py`**
  - Downloads Azure Blob CSV logs.
  - Cleans, clips, log-transforms, and aggregates invocation data into time series.
  - Saves processed dataset + visualization in `outputs/`.
  - Highlights daily & weekly workload patterns with reduced noise from extreme spikes.

### Forecasting Models
- **`prophet_forecast.py`**
  - Trains a Prophet model.
  - Forecasts next 1440 minutes (1 day).
  - Produces plots & CSVs: full predictions vs actuals + next-day forecast.

- **`lstm_forecast.py`**
  - Trains an LSTM with a 60-minute sliding window.
  - Predicts next 1440 minutes.
  - Reports MAE/RMSE, generates visualization + CSVs.
  - Captures short-term spikes better than Prophet.

- **`hybrid.py`**
  - Averages Prophet & LSTM predictions.
  - Computes hybrid MAE/RMSE.
  - Saves full & next-day forecast CSVs.
  - Plot shows Hybrid (purple) vs Prophet (blue) vs LSTM (green) vs Actual (black).

### Reinforcement Learning
- **`autoscaling_env.py`**
  - Custom Gymnasium environment simulating serverless autoscaling.
  - Tracks cold starts, overprovisioning, and rewards based on actions.

- **`ppo_autoscaler.py`**
  - Trains PPO agent on Prophet/LSTM/Hybrid forecasts.
  - Evaluates scaling performance.
  - Exports results (cold starts, overprovision, MAE, RMSE) as CSVs.

### Azure Function Deployment
- **`__init__.py`**
  - Entry point for the Azure Function.
  - Calls `forecast.py` for demand prediction and `ppo.py` for scaling logic.
  - Returns forecast + scaling decisions as JSON API response.

---

## Deployment with Docker & Azure

### Docker
- Project packaged into a Docker image: `hybridautoscaler.az`.
- Includes Prophet, LSTM, PPO agent, and Azure Function code.

### Azure Container Registry (ACR)
- Docker image stored securely in ACR.
- Can be pulled by Azure Functions & other compute services.

### Azure Functions
- Function App deployed from ACR Docker image.
- Runs on Linux App Service Plan.
- Exposes HTTP endpoint for autoscaling API.

---

## Testing the API

In Azure Functions > Code + Test, use the following JSON body:

```json
{
  "current_instances": 5,
  "forecast_periods": 60
}
```

Click Run to trigger the function.
- The API returns forecasted demand + scaling decision (from PPO).
- This validates the hybrid forecasting + RL autoscaler in a live cloud environment.

---

## Evaluation Metrics
- Forecast Accuracy: MAE, RMSE
- Scaling Metrics: Cold starts, overprovision penalties
- Model Comparison: Prophet vs LSTM vs Hybrid vs PPO agent


