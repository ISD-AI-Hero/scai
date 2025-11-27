---

# 🚀 Executive Summary — SWFT Container CI/CD Pipeline

This repository contains a **secure, end-to-end CI/CD pipeline** that builds, scans, signs, analyzes, and assesses a Python FastAPI container for deployment in **Azure Government Cloud**.
The workflow is split into two main jobs:

---

## **1. Build & Security Pipeline (build-and-upload)**

This job automatically runs on **push, PR, or manual workflow dispatch** and performs the following:

### 🔨 **Build & Test**

* Checks out source
* Runs Python tests with coverage
* Builds a Docker image and pushes it to **Azure Container Registry (ACR)**

### 🛡️ **Security Scanning & Metadata**

The following artifacts are generated and signed with **Cosign**:

* **SBOM** (CycloneDX JSON)
* **Trivy vulnerability reports** (JSON + SARIF)
* **CodeQL database** (for later security analysis)
* **run.json** (metadata capturing build ID, commit, image digest, artifact names, etc.)

### ☁️ **Uploads**

Uploads depend on two flags:

| Upload Type        | Controlled By      | Destination                         |
| ------------------ | ------------------ | ----------------------------------- |
| Azure Blob Storage | `upload_to_azure`  | `sboms`, `scans`, `runs` containers |
| GitHub Artifacts   | `upload_artifacts` | Individual artifacts per run        |

---

## **2. Final AI-Assisted Security Assessment (final-assessment)**

This job runs **after build-and-upload completes** and combines all security signals:

* Downloads Trivy + CodeQL artifacts
* Pulls latest **SonarQube** vulnerabilities via API
* Runs **SCAI CTF Engine** to merge findings and generate a final:

  ```
  final_assessment_<project>_<run>.json
  ```

### 📊 Final Output Includes:

* Consolidated vulnerabilities from all tools
* AI-evaluated **risk level (Critical / High / Medium / Low)**
* Determination text explaining the risk
* Links to all uploaded artifacts (if Azure upload enabled)

The final assessment is uploaded to:

* **Azure Blob → scans** container
* **GitHub Artifacts** (if enabled)

---

## 📦 Blob Container Layout

| Container | Purpose                                                   |
| --------- | --------------------------------------------------------- |
| **sboms** | Stores CycloneDX SBOMs                                    |
| **scans** | Stores Trivy, CodeQL DB, SonarQube JSON, Final Assessment |
| **runs**  | Stores run.json (metadata for each pipeline run)          |

