# Cloud VM ML Cluster Project

Developing an end-to-end AI-assisted clustering workflow that transforms raw cloud telemetry into meaningful workload archetypes and actionable cloud optimization insights.

## Business Context

**Atlas Financial Group**, a regional bank, runs its on-demand VM fleet across two regional cloud providers.

A modernisation programme over the last three years moved most workloads from on-premises data centres to the cloud. However, many workloads still use "better safe than sorry" sizing decisions from the migration period.

An internal audit found that:

- ~31% of VMs average below 15% CPU utilization
- This represents an estimated **$8M in annual cloud waste**
- Total annual cloud spend is approximately **$52M**
- The FinOps team aims to reduce the proportion of highly underutilized VMs to **below 12% within two billing cycles**

The challenge is that **average CPU utilization alone cannot determine the correct optimization action**.

For example, a VM averaging 8% CPU could be:

- A forgotten VM that should be decommissioned
- An over-provisioned workload that should be downsized
- A bursty application that occasionally spikes to 80% CPU

The objective is therefore to move beyond simple utilization thresholds and identify **behavioural patterns across the VM fleet**.

## Objective

The objective is to group Atlas's VM fleet into a small number of distinct **behavioural archetypes** based on actual CPU and memory utilization patterns.

These archetypes will provide FinOps with an explainable and defensible foundation for standardized VM sizing and optimization decisions, reducing the need to investigate thousands of VMs individually.

## Prototype Scope

The prototype uses:

| Metric | Scope |
|---|---|
| Observation window | 2 days |
| VMs | ~1,500 |
| Telemetry readings | ~765,000 |
| Monitoring interval | 5 minutes |
| Features | CPU and Memory utilization |

This is sufficient to validate the methodology and determine whether meaningful workload archetypes emerge from behavioural data alone.

### Production Expansion

A production implementation would require:

**More data**
- At least 30 days of telemetry
- Full fleet of ~4,500 VMs
- Weekly utilization patterns
- End-of-month batch workloads
- Seasonal patterns

**More features**
- CPU utilization
- Memory utilization
- Disk I/O
- Network throughput

A VM may appear idle from a CPU perspective while being constrained by another resource. Including additional resource metrics reduces the risk of unsafe sizing recommendations.

The core pipeline, feature engineering approach, and clustering methodology remain the same; production primarily expands the **data volume and feature breadth**.

## Data Dictionary

The dataset consists of four related tables joined primarily through `vm_id` and configuration bucket keys.

### 1. VM Information

**File:** `vm_info.csv`

Contains one record per VM describing its lifecycle and provisioned configuration.

| Attribute | Description |
|---|---|
| `vm_id` | Unique identifier for the VM |
| `subscription_id` | Subscription associated with the VM |
| `deployment_id` | Deployment under which the VM was provisioned |
| `vm_created_ts` | VM creation timestamp (Unix epoch seconds) |
| `vm_deleted_ts` | VM deletion timestamp (Unix epoch seconds); blank values indicate the VM is still active |
| `vm_category` | Workload type: Interactive or Delay-Insensitive |
| `core_count_bucket` | Bucket representing the provisioned vCPU allocation |
| `memory_bucket` | Bucket representing the provisioned memory allocation |

### 2. VM Utilization Time Series

**File:** `vm_utilization_timeseries.csv`

Contains CPU and memory utilization measurements collected every five minutes.

| Attribute | Description |
|---|---|
| `vm_id` | Unique identifier for the VM |
| `timestamp` | Start time of the 5-minute monitoring window (Unix epoch seconds, UTC) |
| `min_cpu_5min` | Minimum CPU utilization during the window |
| `max_cpu_5min` | Maximum CPU utilization during the window |
| `avg_cpu_5min` | Average CPU utilization during the window |
| `min_memory_5min` | Minimum memory utilization during the window |
| `max_memory_5min` | Maximum memory utilization during the window |
| `avg_memory_5min` | Average memory utilization during the window |

### 3. CPU Type Definitions

**File:** `cpu_type_definitions.csv`

Defines the CPU configuration buckets used to categorize VMs based on provisioned vCPU capacity.

| Attribute | Description |
|---|---|
| `core_count_bucket` | Unique CPU configuration bucket |
| `bucket_category` | Descriptive label for the CPU capacity range |
| `min_value` | Lower bound of the vCPU range |
| `max_value` | Upper bound of the vCPU range |
| `representative_value` | Representative vCPU value for the bucket |

### 4. Memory Type Definitions

**File:** `memory_type_definitions.csv`

Defines the memory configuration buckets used to categorize VMs based on provisioned memory capacity.

| Attribute | Description |
|---|---|
| `memory_bucket` | Unique memory configuration bucket |
| `bucket_category` | Descriptive label for the memory capacity range |
| `min_value` | Lower bound of the memory range |
| `max_value` | Upper bound of the memory range |
| `representative_value` | Representative memory value for the bucket |

## Expected Outcome

The project aims to produce a small number of explainable VM behavioural archetypes, with each archetype associated with an appropriate cloud optimization action.

For example:

| Behavioural Archetype | Potential Action |
|---|---|
| Consistently idle | Decommission |
| Consistently underutilized | Downsize |
| Low average but highly bursty | Investigate / cautiously right-size |
| Balanced utilization | Maintain |
| Memory-heavy workload | Memory-focused sizing review |

The final clustering results will provide the foundation for a standardized, fleet-level cloud optimization strategy rather than VM-by-VM investigation.

