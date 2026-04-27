# Encounter Rate Workflow

## Introduction

This workflow helps you measure how often your patrols encounter events of interest, so you can understand where and when wildlife sightings, incidents, or other events occur relative to patrol effort.

**What this workflow does:**
- Downloads **patrol observations** and **patrol events** from EarthRanger for the time range you specify
- Reconstructs patrol trajectories and calculates patrol effort (distance traveled and time spent)
- Joins events to patrols and computes encounter rate as **events per hour** and **events per kilometer** of patrol effort
- Builds a grid-based heatmap showing encounter rate across your area of interest
- Exports patrol trajectories and patrol events as data files (CSV or GeoParquet)
- Optionally generates a Word document report summarizing the analysis

**Who should use this:**
- Conservation managers tracking the effectiveness of ranger patrols
- Researchers analyzing the spatial and temporal distribution of wildlife sightings, illegal activity, or other events relative to patrol coverage
- Anyone who needs to combine **patrols** and **events** stored in EarthRanger into a single encounter rate analysis

---

## Prerequisites

Before using this workflow, you need:

1. **Ecoscope Desktop** installed on your computer
   - If you haven't installed it yet, please follow the installation instructions for Ecoscope Desktop

2. **EarthRanger Data Source** configured in Ecoscope Desktop
   - You must have already set up a connection to your EarthRanger server
   - Your data source should be configured with proper authentication credentials
   - You'll need to know the name of your configured data source (e.g., `"mep_dev"`)

3. **Patrols and Patrol Events** recorded in EarthRanger
   - You need at least one **patrol type** with completed patrols within your selected time range
   - Patrol types can be found at `https://<your-site>.pamdas.org/admin/activity/patroltype/`
   - Event types can be found at `https://<your-site>.pamdas.org/admin/activity/eventtype/`
   - You'll need to know the exact "value" (not display name) of any patrol type or event type you want to filter by — for example `"ecoscope_patrol"` or `"wildlife_sighting_rep"`

---

## Installation

1. Select "Workflow Templates" tab
2. Click "+ Add Template"
3. Copy and paste this URL https://github.com/wildlife-dynamics/encounter-rate and wait for the workflow template to be downloaded and initialized
4. The template will now appear in your available template list

---

## Configuration Guide

### Basic Configuration

#### 1. Workflow Details
Add information that helps you differentiate this workflow run from another.

- **Workflow Name** (required): A descriptive name for this run
  - Example: `"Encounter Rate Workflow"`
- **Workflow Description** (optional): A short description of the analysis
  - Example: `"Analyze patrol encounter rates with events."`

#### 2. Data Source
Select the EarthRanger connection to pull patrols and events from.

- **Data Source** (required): Name of your configured EarthRanger data source
  - Example: `"mep_dev"`

#### 3. Time Range
Choose the period of time to analyze.

- **Since** (required): Start date and time
  - Example: `2015-01-10T00:00:00`
- **Until** (required): End date and time
  - Example: `2015-02-28T23:59:59`
- **Timezone** (optional): The timezone used to interpret your dates and to display times in outputs
  - Example: `Africa/Nairobi (UTC+03:00)` or `UTC (UTC+00:00)`

#### 4. Patrol and Event Types
Filter which patrols and events are included in the analysis.

- **Patrol Types** (required): One or more patrol type values to analyze. Leave empty to include all patrol types.
  - Example: `["ecoscope_patrol"]`
  - Note: Use the patrol type "value" from EarthRanger admin, not the display name
- **Event Types** (required): One or more event type values to include. Leave empty to include all event types.
  - Example: `[]` (all event types) or `["wildlife_sighting_rep"]`

#### 5. Process Patrol Observations
Controls how the data is split into dashboard views and how trajectories are colored.

- **Group Data** (optional): Split the dashboard into multiple views by time period or category
  - Time options: Year (`%Y`), Month (`%B`), Year and Month (`%Y-%m`), Date (`%Y-%m-%d`), Day of week (`%A`), Hour (`%H`), Day of year (`%j`), Day of month (`%d`)
  - Category options: Event Type, Patrol Type, Patrol Subject
  - Note: If left empty, all data appears in a single view
- **Style Trajectory By Category** (required): Which patrol attribute is used to color and label trajectories on the map
  - Options: Patrol Type, Patrol Status, Patrol Subject, Patrol Serial Number
  - Default: `Patrol Subject`

#### 6. Persist Patrol Trajectories
Choose the output format for trajectory data files.

- **Filetypes** (required): Output formats. Select one or more of `csv`, `parquet`
  - Default: `["parquet"]`

#### 7. Persist Patrol Events
Choose the output format for event data files.

- **Filetypes** (required): Output formats. Select one or more of `csv`, `parquet`
  - Default: `["parquet"]`

#### 8. Generate Report
Optional Word document summarizing the analysis.

- **Skip Report Generation** (required): Toggle to skip the Word report
  - Default: `true` (skipped)
  - Note: Set to `false` to include the report in the output
- **Template Path** (required): Path or URL to the `.docx` template with Jinja2 placeholders
  - Default: `"https://raw.githubusercontent.com/wildlife-dynamics/encounter-rate/main/resources/templates/encounter_rate_template.docx"`

### Advanced Configuration

These optional settings provide additional control over your workflow.

#### Patrol Status (under Patrol and Event Types)
Restrict the analysis to patrols with a specific status.

- **Patrol Status**: Filter by patrol status
  - Options: `active`, `overdue`, `done`, `cancelled`
  - Default: `["done"]`

#### Trajectory Segment Filter (under Process Patrol Observations)
Limits on trajectory segment length, duration, and speed used to remove noisy or unrealistic segments.

- **Trajectory Segment Filter**:
  - Defaults: min length `0.001` m, max length `100000` m, min time `1` s, max time `172800` s, min speed `0.01` km/h, max speed `500` km/h
  - Note: Aggressive filters can drop large portions of trajectory and shrink `Total Patrol Km` and `Total Patrol Hours`

#### Process Patrol Events
Filter events by location.

- **Bounding Box**: Only include events with coordinates inside these min/max latitude and longitude values
  - Default: full world extent
- **Filter Exact Point Coordinates**: Drop events recorded at specific coordinates (useful for removing known bad-data points such as `(0, 0)`)
  - Default: `[(180, 90), (0, 0), (1, 1)]`

#### Map Base Layers
Select tile layers to use as base layers in the maps. The first layer in the list is the bottommost layer displayed.

- **Map Base Layers**: Choose from preset layers (Open Street Map, Roadmap, Satellite, Terrain, LandDx, USGS Hillshade) or supply a Custom Layer URL
  - Default: World Topo Map (opacity `1`) with World Imagery overlay (opacity `0.5`)

#### Encounter Rate Map
Controls the grid used to compute the encounter rate heatmap.

- **Grid Cell Size**: Choose `Auto-scale` for an optimized cell size based on your data extent, or `Customize` to set a specific size
  - Default: `Auto-scale`
  - Custom range: greater than `0` and less than `10000` (in the units of the chosen coordinate reference system — meters for the default `EPSG:3857`)
  - Note: Smaller cells give finer detail but more empty cells; larger cells generalize the pattern
- **Coordinate Reference System**: The CRS used for the grid calculation
  - Default: `EPSG:3857`

#### Filename Prefixes
Customize the names of your output files.

- **Persist Patrol Trajectories - Filename Prefix**: Custom prefix for trajectory files
  - Default: `"patrol_trajectories"`
  - Example: `"march_patrols_traj"` will create files like `march_patrols_traj_abc123.parquet`
- **Persist Patrol Events - Filename Prefix**: Custom prefix for event files
  - Default: `"patrol_events"`

---

## Running the Workflow

Once you've configured all the settings:

1. **Review your configuration**
   - Double-check your time range, data source, patrol types, and event types

2. **Save and run**
   - Click "Submit" and the workflow will show up in the "My Workflows" table in Ecoscope Desktop
   - Click on "Run" and the workflow will begin processing

3. **Monitor progress and wait for completion**
   - You'll see status updates as the workflow runs
   - Processing time depends on:
     - The size of your date range
     - The number of patrols and events fetched
     - The grid cell size you chose (smaller cells = more cells to compute)
   - The workflow completes with status "Success" or "Failed"

---

## Understanding Your Results

After the workflow completes successfully, you'll find your outputs in the designated output folder.

### How Encounter Rate Is Calculated

The workflow calculates encounter rates in two complementary ways: **overall summary stats** and a **per-grid-cell heatmap**.

#### Overall Summary Stats (Dashboard Numbers)

Patrol effort is computed first, by summing each patrol's segment distances and durations from its reconstructed trajectory:

- `total_dist_m` per patrol = sum of segment lengths in meters
- `total_time_s` per patrol = sum of segment durations in seconds

The overall stats are then:

- **Total Events** = count of qualifying patrol events
- **Total Patrol Hours** = sum of `total_time_s` across distinct patrols ÷ 3600
- **Total Patrol Km** = sum of `total_dist_m` across distinct patrols ÷ 1000
- **Events Per Hour** = `Total Events × 3600 ÷ sum(total_time_s)`
- **Events Per Km** = `Total Events × 1000 ÷ sum(total_dist_m)`

Each patrol's effort is counted only once even if multiple events occurred during it.

#### Encounter Rate Grid (Heatmap)

The heatmap answers a different question: *where on the map are events most concentrated relative to where patrols actually went?*

1. **Build a meshgrid** over the bounding extent of the patrol events. You choose between:
   - `Auto-scale`: cell size is sized automatically from the data extent
   - `Customize`: you pick a fixed cell size in the CRS units (meters for the default `EPSG:3857`)
2. **For each grid cell**, the workflow:
   - Counts the **events** whose location falls inside the cell → `Events`
   - Calculates the **patrol effort** that intersects the cell — the length of patrol trajectory segments crossing the cell, in kilometers → `Patrol Effort (km)`
   - Computes **Encounter Rate (per km)** = `Events ÷ Patrol Effort (km)`
3. **Cells with no patrol effort or no events are dropped** so they don't dominate the legend
4. **Remaining values are classified into 10 equal-interval bins** and colored using a red-yellow-green reversed colormap (red = high encounter rate, green = low)

This means a hot (red) cell is a place where, *per kilometer that patrols actually walked or drove there*, many events were recorded — not just a place with many events. Areas with a lot of events but a lot of patrol effort can end up cooler than areas with fewer events but very little patrol effort.

### Data Outputs

#### Patrol Trajectories

- **File formats**: CSV and/or GeoParquet (based on your selection)
- **Opens in**: Microsoft Excel, Google Sheets (CSV); Python/R, QGIS/ArcGIS (GeoParquet)
- **Contents**: One row per trajectory segment, with columns including:
  - `patrol_serial_number`: Unique identifier for each patrol
  - `patrol_type`: The patrol type value
  - `patrol_subject`: The subject (ranger or vehicle) doing the patrol
  - `segment_start`: Start time of the segment (in your selected timezone)
  - `dist_meters`: Length of the segment in meters
  - `timespan_seconds`: Duration of the segment in seconds
  - `speed_kmhr`: Average speed of the segment in km/h
  - `geometry`: Line geometry of the trajectory segment

#### Patrol Events

- **File formats**: CSV and/or GeoParquet (based on your selection)
- **Contents**: One row per event, with columns including:
  - `patrol_serial_number`: Identifier of the patrol the event belongs to
  - `patrol_type`: The patrol type value
  - `event_type`: The event type value
  - `time`: Event timestamp (in your selected timezone)
  - `total_dist_m`, `total_time_s`: Patrol effort summary merged onto each event
  - `geometry`: Point geometry of where the event was recorded

### Visual Outputs (Dashboard)

The dashboard combines summary stats and two interactive maps.

#### Summary Stat Widgets (top row)
- **Total Events**: Total event count
- **Total Patrol Hours**: Sum of distinct patrol durations (1 decimal)
- **Events Per Hour**: Encounter rate per hour of patrol (2 decimals)
- **Total Patrol Km**: Sum of distinct patrol distances (1 decimal)
- **Events Per Km**: Encounter rate per kilometer of patrol (2 decimals)

#### Patrol Trajectories and Events Map
- **Format**: Interactive map
- **Layers**:
  - Patrol trajectory lines colored by your selected category (Patrol Type, Patrol Status, Patrol Subject, or Patrol Serial Number)
  - Event points colored by event type
- **Hover tooltips**:
  - Trajectories: Patrol Serial, Patrol Type, Start Time, Duration (s), Speed (kph)
  - Events: Patrol Serial, Event Type, Event Time

#### Encounter Rate Map
- **Format**: Interactive grid-based heatmap
- **Color scale**: Reversed Red-Yellow-Green (red = highest encounter rate, green = lowest), 10 equal-interval bins
- **Hover tooltips**: Encounter Rate (per km), Events, Patrol Effort (km)
- **Legend**: Encounter rate ranges with two-decimal labels

### Grouped Outputs

If you set a **Group Data** value (e.g. monthly, or by patrol type), each widget and map appears as a separate view per group, and you can switch between views in the dashboard.

### Word Document Report (optional)

If **Skip Report Generation** is set to `false`, an additional `.docx` report (`encounter_rate_report_*.docx`) is generated, containing the time range, the summary stats table, and screenshots of the patrol map and encounter rate map for each group.

---

## Common Use Cases & Examples

Here are typical scenarios and how to configure the workflow for each:

### Example 1: Baseline Encounter Rate Analysis
**Goal**: Get the overall encounter rate of all events recorded during ecoscope patrols across a two-month period, with a 5 km grid heatmap.

**Configuration**:
- **Time Range**:
  - Since: `2015-01-10T00:00:00`
  - Until: `2015-02-28T23:59:59`
  - Timezone: `Africa/Nairobi (UTC+03:00)`
- **Patrol Types**: `[]`
- **Event Types**: `[]` (include all)
- **Patrol Status**: `["done"]`
- **Group Data**: empty (single view)
- **Style Trajectory By Category**: `Patrol Subject`
- **Encounter Rate Map - Grid Cell Size**: `Auto-scale`
- **Persist Patrol Trajectories - Filetypes**: `["parquet"]`
- **Persist Patrol Events - Filetypes**: `["parquet"]`
- **Skip Report Generation**: `true`

**Result**:
- One patrol map and one encounter rate map covering the full date range
- Five summary widgets showing totals across the entire period
- Trajectory and event data files in GeoParquet format

---

### Example 2: Grouped Analysis (by Month or by Patrol Type)
**Goal**: Compare encounter rates across multiple subgroups — either by month or by patrol type — to spot temporal trends or differences between patrol programs.

**Configuration A — Monthly views**:
- **Time Range**: `2015-01-10T00:00:00` to `2015-02-28T23:59:59`, `Africa/Nairobi`
- **Group Data**: Time grouper, `Month (%B)`
- **Style Trajectory By Category**: `Patrol Subject`
- **Encounter Rate Map - Grid Cell Size**: `Customize`, `5000`

**Configuration B — Patrol Type views**:
- Same as above, except:
- **Group Data**: Category grouper, `Patrol Type`
- **Style Trajectory By Category**: `Patrol Type`
- **Patrol Types**: `[]` (include all so multiple types appear as separate views)

**Result**:
- A view selector at the top of the dashboard lets you switch between months (`January`, `February`) or between patrol types
- Each view has its own patrol map, encounter rate heatmap, and summary widgets, so you can directly compare rates across groups

---

### Example 3: Choosing a Grid Cell Size (Auto-scale vs. Fine 1 km)
**Goal**: Decide between letting the workflow size the grid for you and using a fine 1 km grid for detailed mapping.

**Configuration A — Auto-scale**:
- **Time Range**: `2015-01-10T00:00:00` to `2015-02-28T23:59:59`, `Africa/Nairobi`
- **Encounter Rate Map - Grid Cell Size**: `Auto-scale`

**Configuration B — 5 km grid**:
- Same as above, except:
- **Encounter Rate Map - Grid Cell Size**: `Customize`, `1000`

**Configuration B — Fine 1 km grid**:
- Same as above, except:
- **Encounter Rate Map - Grid Cell Size**: `Customize`, `1000`

**Result**:
- **Auto-scale** produces a grid size based on the density of the geo events
- **5 or 1 km custom grid** are two common grid sizes used to evaluate patrol effort. A more detailed heatmap that reveals fine-scale hotspots, but may take longer for execution
- Choose smaller cell sizes for small, intensively patrolled areas; larger cells (or `Auto-scale`) for large landscapes

---

### Example 4: Generate a Word Document Report
**Goal**: Produce a shareable `.docx` summary alongside the dashboard, for example to send to stakeholders.

**Configuration**:
- **Time Range**: `2015-01-10T00:00:00` to `2015-02-28T23:59:59`, `Africa/Nairobi`
- **Group Data**: Time grouper, `Month (%B)` (so the report has one section per month)
- **Encounter Rate Map - Grid Cell Size**: `Customize`, `5000`
- **Skip Report Generation**: `false`
- **Template Path**: leave as the default URL, or supply your own `.docx` template

**Result**:
- A Word document `encounter_rate_report_*.docx` in the output folder
- The report contains the report date, an encounter stats table, and screenshots of the patrol map and encounter rate map — one section per group when **Group Data** is set
- Note: report generation adds processing time, since each map needs to be screenshotted

---

## Troubleshooting

### Common Issues and Solutions

#### Workflow Fails to Start
**Problem**: The workflow does not begin running, or it fails immediately.

**Solutions**:
- Verify your EarthRanger data source is configured correctly in Ecoscope Desktop
- Confirm your authentication credentials are still valid
- Check that the data source name in the configuration exactly matches what you set up
- Try refreshing the workflow template

#### No Patrols or Events Returned
**Problem**: The workflow runs but returns empty data, or several tasks are skipped.

**Solutions**:
- Verify your time range — patrols must have `patrol_start_time` overlapping the range, and patrols are filtered by their `Patrol Status` (default: `done`)
- Confirm the **Patrol Types** value(s) match exactly the values in EarthRanger admin (`https://<your-site>.pamdas.org/admin/activity/patroltype/`) — these are values, not display names, and are case-sensitive
- Try leaving **Patrol Types** and **Event Types** empty to include all
- Check that **Patrol Status** isn't filtering out the patrols you want (e.g. `active` vs `done`)

#### Encounter Rate Map Is Empty or Sparse
**Problem**: The grid heatmap has very few colored cells or doesn't appear at all.

**Solutions**:
- If using a custom grid cell size, try a larger value or switch to `Auto-scale`
- Verify the events have valid point geometries — events without geometry are filtered out of the map even if **Include Events Without a Geometry** is `true`
- Check that the **Filter Exact Point Coordinates** list isn't dropping legitimate events
- Confirm your patrols actually overlap the area where events were recorded — cells without patrol effort are removed from the map

#### Workflow Runs Very Slowly
**Problem**: The workflow takes a long time to complete.

**Solutions**:
- Reduce your time range
- Use a larger grid cell size (or `Auto-scale`) — fine grids increase compute time substantially
- Disable **Generate Report** if you don't need the Word document — screenshot generation is one of the slowest steps
- Note: the first run after starting Ecoscope Desktop has a brief warm-up; subsequent runs are faster

#### Wrong Patrols Are Being Colored or Grouped
**Problem**: Map colors or dashboard views don't reflect the dimension you wanted.

**Solutions**:
- Adjust **Style Trajectory By Category** to the attribute you care about (Patrol Type, Patrol Status, Patrol Subject, Patrol Serial Number)
- If you're grouping the dashboard by patrol type, also set **Style Trajectory By Category** to `Patrol Type` for visual consistency
- Verify that your patrols in EarthRanger have the expected values populated for that attribute

#### Encounter Rate Numbers Look Off
**Problem**: Events Per Hour or Events Per Km values seem too high, too low, or zero.

**Solutions**:
- Confirm your **Trajectory Segment Filter** isn't excluding most segments — extreme min/max speed or duration values can drop large portions of the trajectory and shrink `Total Patrol Km` / `Total Patrol Hours`
- Remember the rates use *distinct* patrol effort: an event without a corresponding patrol with effort data will still count toward `Total Events` but contribute nothing to the denominator
- If `Total Patrol Hours` or `Total Patrol Km` is `0`, the per-hour and per-km rates will display as empty — verify the relevant patrols have GPS observations during the time range

#### Report Generation Fails
**Problem**: The workflow succeeds but the `.docx` report is missing or failed to generate.

**Solutions**:
- Confirm **Skip Report Generation** is set to `false`
- Verify the **Template Path** URL or local path is reachable and points to a valid `.docx` file
- Check the workflow logs for screenshot timeouts — very large maps can exceed the 20-second screenshot timeout
- If you supplied a custom template, confirm its Jinja2 placeholders match the keys produced by the workflow (`report_date`, `encounter_stats`, `patrol_maps`, `rate_maps`)
