# LVRoadNetwork2Vec

A Python-based project for converting Las Vegas traffic sensor locations into graph representations, creating edge lists for network analysis, and visualizing road networks using interactive maps. This project enables graph-based analysis of traffic sensor networks using node2vec embeddings and other graph neural network techniques.

## Overview

This project processes traffic sensor data from Las Vegas highways to:
- Clean and normalize sensor location data
- Group sensors by highway/road segments
- Generate edge lists representing sensor connectivity
- Create interactive visualizations using Folium maps
- Enable graph-based machine learning analysis

## Features

- **Data Cleaning**: Remove duplicate sensors and normalize GPS coordinates
- **Sensor Grouping**: Filter and group sensors by highway identifiers (I-15, US-95, CC-215, I-515)
- **Edge List Generation**: Two methods for creating graph edges:
  - Minimum Spanning Tree (MST) based on physical distances
  - Sequential adjacency for ordered sensor networks
- **Interactive Visualization**: Color-coded maps showing sensor locations and connections
- **Multi-Highway Support**: Visualize multiple highway networks simultaneously

## Installation

### Prerequisites

- Python 3.7+
- pip package manager

### Required Libraries

```bash
pip install pandas
pip install folium
pip install geopy
pip install jupyter
```

Or install all at once:

```bash
pip install pandas folium geopy jupyter
```

## Project Structure

```
LVRoadNetwork2Vec/
├── README.md                           # Project documentation
├── edge_list_maker.ipynb              # Main notebook for edge list creation
├── plotter.ipynb                      # Visualization notebook
├── detector/                          # Traffic sensor data
│   ├── bugatti/                       # BUGATTI system data
│   │   ├── FAST Detectors 2021.09.23 - XIE.xlsx
│   │   └── sensor_description_all.csv
│   └── shredder (UNLV)/              # UNLV detector data
│       ├── detectors2018.csv
│       └── detectors2019.csv
└── graphics/                          # Output visualizations
```

## Data Format

The CSV sensor data files contain the following columns:

| Column | Description | Example |
|--------|-------------|---------|
| 0 | Index/ID | - |
| 1 | Detector/Sensor ID with lane info | `I15_NB_SAHARA_lane1` |
| 2 | - | - |
| 3 | Location description | `I-15 NB @ Sahara` |
| 4 | - | - |
| 5 | Latitude (compressed format) | `3617` → 36.17° |
| 6 | Longitude (compressed format) | `11513` → -115.13° |

## Usage

### 1. Basic Sensor Visualization

Use `plotter.ipynb` to visualize all sensors:

```python
import folium
import pandas as pd

# Load and process data
data = pd.read_csv('./detector/shredder (UNLV)/detectors2019.csv', header=None)
latitude = data.iloc[:, 5].astype(str).apply(lambda x: float(x[:2] + '.' + x[2:]))
longitude = data.iloc[:, 6].astype(str).apply(lambda x: float(x[:4] + '.' + x[4:]))

# Create map
map = folium.Map(location=[latitude.mean(), longitude.mean()], zoom_start=10)
# Add markers...
```

### 2. Create Edge Lists

Use `edge_list_maker.ipynb` for graph generation:

```python
# Load and clean data
data = clean_dataset(pd.read_csv('./detector/shredder (UNLV)/detectors2018.csv', header=None))

# Filter specific highway
sensor_loc = "I-15 NB"  # Options: "I-15 NB", "I-15 SB", "US-95 NB", "US-95 SB", "CC-215 EB", "CC-215 WB", "I-515 SB", "I-515 NB"
sensor_group = get_sensor_group(data, sensor_loc, ignore='None')

# Generate edge list and visualization
map, edge_list = tsp(sensor_group)
display(map)
print(edge_list)
```

### 3. Multi-Highway Visualization

Visualize all highways with color coding:

```python
# Run main() function in edge_list_maker.ipynb
# Automatically generates a map with:
# - Red: CC-215 (East/West)
# - Green: I-15 (North/South)
# - Purple: I-515 (North/South)
# - Yellow: US-95 (North/South)
```

## Core Functions

### `clean_dataset(df)`
Cleans and normalizes the sensor dataset.

**Parameters:**
- `df` (DataFrame): Raw sensor data

**Returns:**
- DataFrame: Cleaned data with:
  - Duplicate sensors removed (based on lat/long)
  - Lane identifiers stripped from sensor names
  - GPS coordinates converted to decimal format

**Example:**
```python
data = pd.read_csv('./detector/shredder (UNLV)/detectors2018.csv', header=None)
clean_data = clean_dataset(data)
```

### `get_sensor_group(data, sensor_loc, ignore=None)`
Filters sensors by highway identifier.

**Parameters:**
- `data` (DataFrame): Cleaned sensor data
- `sensor_loc` (str): Highway identifier (e.g., "I-15 NB", "US-95 SB")
- `ignore` (str, optional): String pattern to exclude from results

**Returns:**
- DataFrame: Filtered sensor group

**Example:**
```python
i15_sensors = get_sensor_group(data, "I-15 NB", ignore="-")
```

### `calculate_distance(sensor_data)`
Calculates pairwise distances between all sensors using GPS coordinates.

**Parameters:**
- `sensor_data` (DataFrame): Sensor group data

**Returns:**
- list[list[float]]: Distance matrix in meters

### `tsp(sensor_data)` - Method 1: MST-based

Creates edge list using Minimum Spanning Tree algorithm based on physical distances.

**Parameters:**
- `sensor_data` (DataFrame): Sensor group data

**Returns:**
- `map` (folium.Map): Interactive map with sensor connections
- `edge_list` (DataFrame): Edge list with columns [sensor1, sensor2, cost]

**Example Output:**
```
  sensor1         sensor2         cost
  I15_NB_SAHARA  I15_NB_CHARLESTON  1247.3
  I15_NB_OAKEY   I15_NB_SAHARA      891.2
```

### `tsp(sensor_data)` - Method 2: Sequential Adjacency

Creates 0-1 edge list where consecutive sensors are connected.

**Returns:**
- `map` (folium.Map): Interactive map
- `edge_list` (DataFrame): Edge list with columns [sensor1, sensor2, weight]
  - `weight=1`: Adjacent sensors
  - `weight=0`: Non-adjacent sensors

**Example Output:**
```
  sensor1  sensor2  weight
  1        2        1
  1        3        0
  2        3        1
```

## Highway Identifiers

The following highway segments are available in the dataset:

- **I-15**: Interstate 15 (Northbound/Southbound)
- **US-95**: US Route 95 (Northbound/Southbound)
- **CC-215**: Clark County 215 Beltway (Eastbound/Westbound)
- **I-515**: Interstate 515 (Northbound/Southbound)

## Applications

This project enables various graph-based analyses:

1. **Node2Vec Embeddings**: Generate vector representations of sensors for ML models
2. **Traffic Flow Analysis**: Study traffic patterns across connected sensors
3. **Network Coverage**: Analyze sensor distribution and identify coverage gaps
4. **Graph Neural Networks**: Train GNNs on the sensor network topology
5. **Anomaly Detection**: Identify unusual traffic patterns using graph structure

## Output Files

- **Interactive Maps**: HTML files saved using `map.save('output.html')`
- **Edge Lists**: CSV files with sensor connectivity
- **Visualizations**: Stored in `graphics/` directory

## Contributing

Contributions are welcome! Please feel free to submit pull requests or open issues for bugs and feature requests.

## Data Sources

- **UNLV Shredder Data**: 2018-2019 traffic detector data
- **BUGATTI System**: FAST Detectors data (2021)

## License

MIT License

## Citation

If you use this project in your research, please cite:

```
T. Bin Zahid and B. T. Morris, "Benchmarking/Limitations of Traffic Prediction with Noisy Field Measurements," 2024 IEEE International Conference on Vehicular Electronics and Safety (ICVES), Ahmedabad, India, 2024, pp. 1-6, doi: 10.1109/ICVES61986.2024.10928136. keywords: {Training;Vehicular and wireless technologies;Accuracy;Roads;Urban planning;Predictive models;Transformers;Data models;Robustness;Noise measurement},

T. b. Zahid and B. Morris, "Using Deep Traffic Prediction for EMFAC Emission Estimation and Visualization," 2024 IEEE 27th International Conference on Intelligent Transportation Systems (ITSC), Edmonton, AB, Canada, 2024, pp. 2488-2493, doi: 10.1109/ITSC58415.2024.10919675. keywords: {Solid modeling;Accuracy;Decision making;Transportation;Estimation;Data visualization;Predictive models;Transformers;Data models;Environmental factors},



```

---

**Last Updated**: January 2026
