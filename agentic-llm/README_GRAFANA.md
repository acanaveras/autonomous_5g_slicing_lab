# 5G Network Agent: Grafana & InfluxDB Integration Guide

This guide explains how to set up and use the enhanced real-time metrics dashboard for the 5G Network Agent, using **Grafana** and **InfluxDB** for professional, interactive visualization. It is tailored for users starting from the original [NVIDIA/GenerativeAIExamples](https://github.com/NVIDIA/GenerativeAIExamples/tree/main) repository, and reflects all recent changes and improvements.

---

## 🚀 Overview: What's New

- **Grafana Dashboards**: Interactive, real-time time-series visualizations
- **InfluxDB**: High-performance time-series database for metrics storage
- **Automated Docker Setup**: One-command startup for Grafana & InfluxDB
- **Streamlit UI**: Embedded Grafana dashboard in the main app
- **Test & Utility Scripts**: Easy verification and troubleshooting

---

## 📁 File Structure (Grafana Integration)

```
agentic-llm/
├── chatbot_DLI.py                # Main Streamlit app (Grafana embedded)
├── influxdb_utils.py             # InfluxDB client utility (metrics API)
├── test_influxdb.py              # Script to test InfluxDB connectivity
├── docker-compose.yml            # Docker Compose for Grafana & InfluxDB
├── start_grafana_services.sh     # Linux/Mac startup script
├── start_grafana_services.bat    # Windows startup script
├── requirements_grafana.txt      # Python dependencies
├── grafana/
│   ├── provisioning/
│   │   ├── datasources/
│   │   │   └── influxdb.yaml     # InfluxDB datasource config
│   │   └── dashboards/
│   │       └── dashboard.yaml    # Dashboard provisioning config
│   └── dashboards/
│       └── 5g-metrics-dashboard.json  # Dashboard definition (edit here)
├── config.yaml                   # App configuration (log file paths, etc.)
└── README_GRAFANA.md             # This guide
```

---

## 🛠️ Setup Instructions

### 1. Prerequisites
- **Docker & Docker Compose**: Required for running Grafana and InfluxDB
- **Python 3.8+**

### 2. Install Python Dependencies

```bash
pip install -r requirements_grafana.txt
```

### 3. Start Grafana & InfluxDB Services

**On Linux/Mac:**
```bash
chmod +x start_grafana_services.sh
./start_grafana_services.sh
```

**On Windows:**
```cmd
start_grafana_services.bat
```

This will:
- Stop any existing containers
- Start Grafana (http://localhost:9002) and InfluxDB (http://localhost:9001)
- Provision the dashboard and datasource automatically

### 4. Verify Services
- **Grafana**: [http://localhost:9002](http://localhost:9002) (Username: `admin`, Password: `admin`)
- **InfluxDB**: [http://localhost:9001](http://localhost:9001)

### 5. Test InfluxDB Connection (Optional)
Run the test script to verify InfluxDB is working and pre-populate with sample data:

```bash
python test_influxdb.py
```

---

## 🏗️ Architecture

```
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│ Streamlit UI  │    │   InfluxDB    │    │   Grafana     │
│ (chatbot_DLI) │──▶ │ (metrics DB)  │ ◀──│ (dashboard)   │
│ - Log display │    │ - Store data  │    │ - Real-time   │
│ - Controls    │    │               │    │   charts      │
│ - Grafana     │    │               │    │ - Interactive │
│   embed       │    │               │    │   panels      │
└───────────────┘    └───────────────┘    └───────────────┘
```

---

## 📈 Dashboard Features

- **UE1/UE3 Packet Loss & Transfer Rate**: Real-time monitoring
- **Auto-refresh**: 5-second updates
- **Interactive Panels**: Zoom, pan, time range selection
- **Dark Theme**: Consistent with app
- **Customizable**: Edit `grafana/dashboards/5g-metrics-dashboard.json`

---

## 🔧 Configuration Details

### Grafana
- **URL**: http://localhost:9002
- **Username**: admin
- **Password**: admin
- **Dashboard**: 5G Network Metrics Dashboard

### InfluxDB
- **URL**: http://localhost:9001
- **Organization**: 5g-lab
- **Bucket**: 5g-metrics
- **Token**: 5g-lab-token

### Docker Compose
- Ports: Grafana (9002→3000), InfluxDB (9001→8086)
- Data is persisted in Docker volumes
- All provisioning is automatic via `grafana/provisioning/`

---

## 📊 Data Flow

1. **Data Collection**: Metrics from KineticaDB (via `chatbot_DLI.py`)
2. **Processing**: Data cleaning/validation in app
3. **Storage**: Metrics written to InfluxDB (see `influxdb_utils.py`)
4. **Visualization**: Grafana queries InfluxDB, displays in dashboard
5. **UI Integration**: Dashboard embedded in Streamlit app

---

## 📝 API Reference: InfluxDBMetricsClient

```python
from influxdb_utils import InfluxDBMetricsClient

# Initialize and connect
client = InfluxDBMetricsClient()
client.connect()

# Write a single metric
timestamp = datetime.utcnow()
client.write_metrics(ue="UE1", loss_percentage=2.5, bitrate=45.2, timestamp=timestamp)

# Write a DataFrame
client.write_dataframe(df, ue_column="ue", loss_column="loss_percentage", bitrate_column="bitrate", timestamp_column="timestamp")

# Close connection
client.close()
```

---

## 🧪 Testing & Troubleshooting

### Test InfluxDB
- Run `python test_influxdb.py` to verify connection and write test data.
- Check output for success/failure messages.

### Common Issues

#### Grafana Not Accessible
```bash
docker-compose ps
docker-compose logs grafana
docker-compose restart
```

#### Data Not Appearing
- Ensure InfluxDB is running and reachable
- Check if data is being written (InfluxDB logs)
- Confirm dashboard queries and time range
- Use `test_influxdb.py` to inject test data

#### Performance
- Increase InfluxDB memory in `docker-compose.yml` if needed
- Adjust Grafana refresh interval in dashboard settings

---

## 🎯 Extending the Dashboard

1. Edit `grafana/dashboards/5g-metrics-dashboard.json` to add/modify panels
2. Update queries or visualizations as needed
3. Dashboard is auto-provisioned on container restart

---

## 📄 License

This project is licensed under the Apache 2.0 License (see LICENSE.md). 