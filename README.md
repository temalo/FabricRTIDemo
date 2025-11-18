# FabricRTIDemo

A repository to house artifacts related to a Fabric Real-Time Intelligence (RTI) Demo for monitoring transformer data streams using Microsoft Fabric.

## Overview

This demo showcases a real-time data streaming and analytics solution built on Microsoft Fabric. It simulates a power grid monitoring system that tracks transformer performance metrics, identifies overloaded transformers, and provides real-time dashboards for monitoring equipment health.

The solution demonstrates the integration of multiple Microsoft Fabric components:
- **Lakehouse**: Storage for reference data and streaming data
- **Eventhouse**: Real-time analytics with KQL Database
- **Eventstream**: Data ingestion from Event Hub
- **Notebook**: Spark-based data stream emulator
- **Real-Time Dashboard**: Interactive visualization of transformer metrics

## Repository Structure

```
FabricRTIDemo/
├── Dashboard/              # Real-Time Dashboard configuration
│   └── dashboard-TransformerStream.json
├── Instructions/           # Setup documentation
│   └── RealTimeDashboardDemo.pdf
├── Notebook/              # Spark Notebook for data emulation
│   └── TransformerStreamEmulator.txt
├── Queryset/              # KQL queries for data analysis
│   └── TransformerStream.txt
└── ReferenceFiles/        # Sample transformer data
    ├── Transformers.csv
    └── Transformers_EastUS.csv
```

## Prerequisites

Before setting up this demo, ensure you have:

- A Microsoft Fabric workspace with appropriate permissions (Owner or full Write access)
- A Fabric Capacity (F16 minimum)
- Access to create and manage Fabric resources:
  - Lakehouse
  - Eventhouse
  - Eventstream
  - Notebook
  - Real-Time Dashboard

## Initial Setup

Follow these high-level steps to configure the demo environment:

1. Create a Lakehouse and upload reference data
2. Create an Eventhouse with KQL Database and Queryset
3. Create an Eventstream with appropriate sinks
4. Create a Notebook to simulate data streaming
5. Create a Real-Time Dashboard for visualization
6. (Optional) Create a Power BI Report

## Detailed Setup Instructions

### 1. Create a Lakehouse

1. Navigate to your Fabric workspace
2. Select **New Item** → **Lakehouse**
3. Give the Lakehouse a meaningful name (e.g., "TransformerLakehouse")
4. Click **Create**

#### Upload Reference Files

1. Navigate to your Lakehouse
2. Select the **Files** section
3. From the ellipse menu, select **Upload Files**
4. Upload the following files from the `ReferenceFiles/` folder:
   - `Transformers.csv`
   - `Transformers_EastUS.csv`
5. For each uploaded file:
   - Select the ellipse next to the file
   - Choose **Load to Table** → **New Table**
   - Name the table to match the file name (without extension)

### 2. Create an Eventhouse

1. In your workspace, select **New Item** → **Eventhouse**
2. Give the Eventhouse a meaningful name (e.g., "TransformerEventhouse")
3. Click **Create**
4. Within the Eventhouse, create a **KQL Database**

#### Create Shortcuts to Reference Data

1. Navigate to the **Shortcuts** icon in the KQL database
2. Select **New** → **OneLake Shortcut**
3. Choose **Microsoft OneLake**
4. Navigate to the Lakehouse you created
5. Select the two reference tables:
   - `Transformers`
   - `Transformers_EastUS`

### 3. Create an Eventstream

1. Navigate to your workspace
2. Select **New Item** → **Eventstream**
3. Name the Eventstream (e.g., "TransformerStream")
4. Click **Create**

#### Configure Eventstream Sources and Sinks

1. On the blank canvas, choose **Custom Endpoint**
2. In the Custom Endpoint dialog:
   - Name the source: `TransformerStream`
   - Click **Add**

3. **Add Lakehouse Destination:**
   - Select **Transform events** → **Lakehouse**
   - Name: `TransformerStream`
   - Select your Lakehouse
   - Choose **New Table**
   - Table name: `TransformerStream`
   - Click **Done**

4. **Add KQL Database Destination:**
   - Select **Add Destination**
   - Name: `TransformerStreamEH`
   - Select your Eventhouse
   - Select your KQL Database
   - Create a new table: `ActiveTransformers`
   - Click **Done**
   - Draw a line connecting the stream to this destination

5. Click **Publish** to save and activate the Eventstream

#### Get Connection String

1. Select the input node on the left side of the canvas
2. Select the **Event Hub** tab
3. Choose **SAS Key Authentication** option
4. Copy the **Connection String Primary Key** (you'll need this for the Notebook)

### 4. Create a Notebook

1. Navigate to your workspace
2. Select **New Item** → **Notebook**
3. Give the Notebook a meaningful name

#### Configure Notebook Environment

1. Select the **Environment** option
2. Click the **Edit** button next to the Environment name
3. Select **Public Library** → **Add From PyPI**
4. Add the `azure-servicebus` library
5. Save the environment

#### Add Notebook Code

1. Copy the code from `Notebook/TransformerStreamEmulator.txt`
2. Paste it into your Notebook
3. Locate the Service Bus connection string section:
   ```python
   myconnectionstring = "<Insert Connection String Here for the Eventhouse Servicebus Connection>"
   ```
4. Replace with the connection string you copied from the Eventstream setup
5. Click **Run All** to start the data stream emulation

### 5. Populate the Queryset

1. Navigate to your Eventhouse
2. Select the KQL Database → **Queryset**
3. Copy the content from `Queryset/TransformerStream.txt`
4. Replace the text in the Queryset with the copied queries
5. Test queries by highlighting each query element and selecting **Run** (or Shift+Enter)

### 6. Create a Real-Time Dashboard

1. Navigate to your workspace
2. Select **New Item** → **Real-Time Dashboard**
3. Name the Dashboard: `TransformerStreamDashboard`
4. When it loads, select the **Manage** tab
5. Choose **Replace with File**
6. Select the `Dashboard/dashboard-TransformerStream.json` file

#### Configure Data Sources

After importing the dashboard:
1. Select the **Data sources** button
2. Update the Data source to point to your Eventhouse and KQL database
3. Save the changes

## Usage

Once the setup is complete:

1. **Start the Notebook** to begin streaming simulated transformer data
2. **Monitor the Eventstream** to verify data is flowing to both Lakehouse and KQL Database
3. **View the Real-Time Dashboard** to see:
   - Average power consumption by city
   - Overloaded transformers on a map
   - KVA rating alerts (high and low)
   - Low side voltage alerts
   - Service point data table

## Data Flow

```
Notebook (Data Emulator)
    ↓
Event Hub (via Service Bus)
    ↓
Eventstream
    ├→ Lakehouse (TransformerStream table)
    └→ Eventhouse (ActiveTransformers table)
         ↓
    KQL Queries (join with reference data)
         ↓
    Real-Time Dashboard
```

## Key Features

- **Real-time data ingestion**: Simulates streaming transformer telemetry data
- **Reference data integration**: Joins streaming data with static reference tables
- **Alert detection**: Identifies transformers operating outside normal parameters
- **Geographic visualization**: Maps overloaded transformers by location
- **Historical analysis**: Tracks trends over time using KQL queries

## Notes

- The connection string in the Notebook should be stored securely (e.g., Azure Key Vault) in production environments
- Minimum Fabric Capacity of F16 is required for optimal performance
- The demo uses simulated data; adapt the Notebook code for real data sources
- Adjust time windows in KQL queries based on your demo duration

## License

See the [LICENSE](LICENSE) file for details.
