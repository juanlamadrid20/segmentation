# Customer Segmentation Workflow Diagram

```mermaid
graph TD
    subgraph "Jobs"
        DSJ[Data Setup Job]
        DSJ_T1[generate_synthetic_data<br/>01_Data_Setup.py]
        DSJ --> DSJ_T1

        IJ[Business Insights Job]
        IJ_T1[create_business_insights<br/>03_Business_Insights.py]
        IJ --> IJ_T1

        SMJ[Unsupervised Customer<br/>Segmentation Job]
        SMJ_T1[unsupervised_customer_segmentation<br/>02b_Segmentation_MLflow.py]
        SMJ --> SMJ_T1
    end

    subgraph "Pipeline"
        SP[Segmentation Pipeline<br/>DLT Advanced/Serverless]
        SP_N1[02a_Segmentation_Lakeflow.py]
        SP --> SP_N1
    end

    subgraph "Complete Demo Workflow"
        CSD[Customer Segmentation<br/>Complete Job]

        T1[setup_data<br/>runs Data Setup Job]
        T2[run_segmentation_pipeline<br/>runs DLT Pipeline]
        T3[unsupervised_segmentation<br/>runs MLflow Job]
        T4[generate_insights<br/>runs Insights Job]

        CSD --> T1
        T1 -->|depends_on| T2
        T2 -->|depends_on| T3
        T2 -->|depends_on| T4
    end

    subgraph "Dashboard"
        DASH[Customer Segmentation<br/>Dashboard<br/>customer_segmentation.lvdash.json]
    end

    T1 -.->|triggers| DSJ
    T2 -.->|triggers| SP
    T3 -.->|triggers| SMJ
    T4 -.->|triggers| IJ

    SP_N1 -.->|writes to| DB[(Unity Catalog<br/>catalog: juan_dev<br/>schema: user_segmentation)]
    DSJ_T1 -.->|writes to| DB
    SMJ_T1 -.->|reads from| DB
    IJ_T1 -.->|reads from| DB
    DASH -.->|queries| DB

    style CSD fill:#e1f5ff,stroke:#0288d1,stroke-width:3px
    style DSJ fill:#fff3e0,stroke:#f57c00
    style IJ fill:#fff3e0,stroke:#f57c00
    style SMJ fill:#fff3e0,stroke:#f57c00
    style SP fill:#f3e5f5,stroke:#7b1fa2
    style DASH fill:#e8f5e9,stroke:#388e3c
    style DB fill:#fff9c4,stroke:#f9a825
```

## Workflow Components

### 1. Individual Jobs
- **Data Setup Job**: Generates synthetic customer data
- **Business Insights Job**: Creates business insights from segmented data
- **Unsupervised Segmentation Job**: Runs MLflow-based customer segmentation

### 2. Pipeline
- **Segmentation Pipeline**: Delta Live Tables (DLT) pipeline that processes customer data
  - Edition: Advanced
  - Mode: Serverless
  - Trigger: Manual

### 3. Complete Demo Workflow
The main orchestration job that runs all components in sequence:
1. **setup_data**: Generates synthetic data
2. **run_segmentation_pipeline**: Processes data through DLT pipeline (full refresh)
3. **unsupervised_segmentation**: Runs ML segmentation (depends on pipeline)
4. **generate_insights**: Creates business insights (depends on pipeline)

### 4. Dashboard
- **Customer Segmentation Dashboard**: Visualizes the results from the pipeline and insights

### 5. Data Storage
All components read/write to Unity Catalog:
- Catalog: `juan_dev`
- Schema: `{user}_segmentation`
- Warehouse ID: `8baced1ff014912d`

## Execution Flow
```
Data Setup → DLT Pipeline → [Unsupervised Segmentation + Business Insights] → Dashboard
```
