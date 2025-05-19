# Credit‑Scoring EU AI Act Demo

> A ZenML‑powered end‑to‑end credit‑scoring workflow that automatically generates the technical evidence required by the EU AI Act.

## 🚀 Project Overview

The project implements three main pipelines:

1. **Feature Engineering Pipeline**: Handles data governance and preprocessing (Articles 10, 12, 15)
   – `ingest → data_splitter → data_preprocessor → generate_compliance_metadata`

2. **Training Pipeline**: Implements model training, evaluation, and risk assessment (Articles 9, 11, 15)  
   – `train_model → evaluate_model → risk_assessment`

3. **Deployment Pipeline**: Manages human oversight, deployment, and monitoring (Articles 14, 17, 18)
   – `approve_deployment → modal_deployment → generate_sbom → post_market_monitoring → generate_annex_iv_documentation`

Each run automatically versions its inputs, logs hashes & metrics, and generates a complete Annex IV draft with all required compliance artifacts. These artifacts include an SBOM (Software Bill of Materials), monitoring plan, data profiling reports, risk assessments, and technical documentation.

## Architecture

![End-to-End Architecture](assets/e2e.png)

For detailed diagrams of each pipeline and their compliance mapping, see [Pipeline Diagrams](assets/diagrams.md).

## Project Structure

```bash
credit_scoring_ai_act/
├── app/ # Modal deployment app
│   ├── modal_deployment.py # Modal deployment script
│   └── schemas.py # Pydantic models for API
├── src/
│   ├── pipelines/
│   │   ├── feature_engineering.py # Feature engineering pipeline
│   │   ├── training.py # Model training pipeline
│   │   └── deployment.py # Deployment pipeline
│   ├── steps/
│   │   ├── feature_engineering/ # Feature engineering steps
│   │   │   ├── ingest.py # Load CSV → log SHA‑256, WhyLogs profile
│   │   │   ├── data_preprocessor.py # Basic feature engineering
│   │   │   ├── data_splitter.py # Split dataset into train/test
│   │   │   └── generate_compliance_metadata.py # Generate compliance metadata
│   │   ├── training/ # Training steps
│   │   │   ├── train.py # XGBoost / sklearn model
│   │   │   ├── evaluate.py # Standard + Fairness metrics
│   │   │   └── risk_assessment.py # Risk assessment
│   │   └── deployment/ # Deployment steps
│   │       ├── approve.py # Human‑in‑loop gate (approve_deployment step)
│   │       ├── deploy.py # Deployment to Modal
│   │       ├── post_market_monitoring.py # Post‑market monitoring
│   │       ├── generate_sbom.py # Generate Software Bill of Materials
│   │       └── post_run_annex.py # Generate Annex IV documentation
│   ├── utils/ # Shared utilities
│   │   ├── modal_utils.py # Modal Volume operations
│   │   ├── preprocess.py # Custom sklearn transformers
│   │   ├── eval.py # Evaluation utils
│   │   ├── incidents.py # Incident reporting system
│   │   ├── risk_dashboard.py # Risk visualization dashboard
│   │   ├── visualizations.py # Visualization utils
│   │   └── model_definition.py # ZenML model definition
│   │
│   ├── configs/ # Configuration files
│   └── constants.py # Centralized configuration constants
│
├── docs/
│   ├── risk/ # Risk assessment documentation
│   │   ├── incident_log.json # Incident tracking
│   │   └── risk_register.xlsx # Risk register
│   ├── releases/ # Compliance artifacts organized by run ID
│   │   └── <run_id>/
│   │      ├── annex_iv_cs_deployment.md  # Annex IV technical documentation
│   │      ├── evaluation_results.yaml    # Model performance metrics and evaluations
│   │      ├── git_info.md                # Git commit and repository information
│   │      ├── monitoring_plan.json       # Model monitoring configuration
│   │      ├── README.md                  # Release-specific information
│   │      ├── risk_scores.yaml           # Risk assessment scores and analysis
│   │      ├── sbom.json                  # Software Bill of Materials
│   │      └── whylogs_profile.html       # Data profiling report
│   └── templates/
│       ├── annex_iv_template.j2 # Annex IV template
│       ├── sample_inputs.json # Sample inputs for Annex IV
│       └── qms/ # Quality management system documentation
│           ├── qms_template.md # Core QMS document
│           ├── roles_and_responsibilities.md # Role assignments
│           ├── audit_plan.md # Audit schedule and methodology
│           └── sops/ # Standard Operating Procedures
│               ├── model_release_sop.md # Model release protocol
│               ├── drift_monitoring_sop.md # Monitoring procedures
│               ├── incident_response_sop.md # Incident handling
│               ├── risk_mitigation_sop.md # Risk management process
│               └── data_ingestion_sop.md # Data handling procedures
│
├── assets/ # Pipeline diagrams and documentation
│   ├── deployment-pipeline.png # Deployment pipeline diagram
│   ├── diagrams.md # Detailed pipeline diagrams with explanations
│   ├── e2e.png # End-to-end architecture diagram
│   ├── feature-engineering-pipeline.png # Feature engineering pipeline diagram
│   ├── modal-deployment.png # Modal deployment diagram
│   └── training-pipeline.png # Training pipeline diagram
├── data/ # Dataset directory
│   └── credit_scoring.csv # Credit scoring dataset
├── visualizations/ # WhyLogs and other visualization outputs
├── run.py # CLI entrypoint
├── COMPLIANCE.md # EU AI Act compliance mapping
└── README.md
```

## Running Pipelines

```bash
# Run feature engineering pipeline (Articles 10, 12)
python run.py --feature

# Run model training pipeline (Articles 9, 11, 15)
python run.py --train

# Run deployment pipeline (Articles 14, 17, 18)
python run.py --deploy
```

Options:

- `--auto-approve` for non‑interactive deployment
- `--no-cache` to disable ZenML caching
- `--config-dir <path>` to override default configs

### Configuration

Pipeline configurations are stored in the `src/configs/` directory:

- `feature_engineering.yaml`
- `training.yaml`
- `deployment.yaml`

You can specify a custom config directory using the `--config-dir` option.

## Modal Deployment

![Modal Deployment](assets/modal-deployment.png)

The project implements a serverless deployment using Modal with comprehensive monitoring and incident reporting capabilities:

- FastAPI application with documented endpoints
- Automated model and preprocessing pipeline loading
- Drift detection and incident reporting
- Standardized storage paths for compliance artifacts
- Complete incident reporting system for Article 18 compliance

## 🔗 EU AI Act Compliance Mapping

For a complete overview of the EU AI Act compliance mapping, refer to the [COMPLIANCE.md](COMPLIANCE.md) file.

## Compliance Directory Structure

The repository uses a structured approach to organizing compliance artifacts:

| Directory               | Purpose                                  | Auto/Manual |
| ----------------------- | ---------------------------------------- | ----------- |
| **releases/**           | Compliance artifacts organized by run ID | Auto        |
| **risk/**               | Risk assessment and incident tracking    | Auto/Manual |
| **templates/**          | Templates for document generation        | Manual      |
| **templates/qms/**      | Quality Management System documentation  | Manual      |
| **templates/qms/sops/** | Standard Operating Procedures            | Manual      |

The **releases/** directory contains automatically generated artifacts for each pipeline run, including:

- Annex IV technical documentation (annex_iv_cs_deployment.md)
- Software Bill of Materials (sbom.json)
- Model performance metrics (evaluation_results.yaml)
- Risk assessment scores (risk_scores.yaml)
- Data profiling report (whylogs_profile.html)
- Monitoring configuration (monitoring_plan.json)
- Git repository information (git_info.md)

## 📋 Quality Management System (Article 17)

This repo delivers all _technical evidence_ (lineage, metadata, logs) required by the EU AI Act. For a complete QMS, you must also maintain formal QMS documentation.

See `docs/templates/qms/` for starter templates:

- **Quality Policy** (`qms_template.md`)
- **Roles & Responsibilities** (`roles_and_responsibilities.md`)
- **Audit Plan** (`audit_plan.md`)
- **SOPs** (`sops/` folder: data ingestion, model release, risk mitigation, etc.)
