# # End-to-End DevSecOps Pipeline: Jenkins + Vault + Snyk + SonarQube + JFrog + Jira

This repository contains a secure CI/CD pipeline leveraging Vault AppRole, AWS STS, and full-spectrum security scans (IaC, SAST, binary, DAST) with automatic Jira ticketing and artifact promotion.


This repository contains a Jenkins pipeline designed to automate secure log ingestion, scanning, and artifact handling, with integrations across Vault, SonarQube, Snyk, JFrog, and Jira. It is designed for extensibility, and upcoming enhancements include AWS Athena integration for log querying, and Dead Letter Queues (DLQs) for enhanced fault tolerance in Lambda processing and AWS Security Hub events.

---

## 🔐 Features

- **Secrets Management**: Retrieves credentials and tokens securely using HashiCorp Vault (AppRole).
- **Code Checkout**: Pulls code from GitHub.
- **Static Code Analysis**: SonarQube scan with automatic Jira ticket creation for high-severity issues.
- **IaC Security Scanning**: Snyk scans Infrastructure-as-Code for misconfigurations.
- **Artifact Creation & Upload**: Simulated artifact creation, uploaded to JFrog Artifactory.
- **Binary Vulnerability Scanning**: JFrog Xray scan of uploaded artifact.
- **Jira Integration**: Automatically creates detailed Jira issues for findings.
- **Planned Enhancements**:
  - 🧠 **Athena** for log analysis and querying
  - 🔁 **Dead Letter Queues (DLQs)** for Lambda (log processing failures)
  - 🛡️ **Security Hub DLQ** for secure triaging of unprocessed findings

---

## 📦 Pipeline Breakdown

### 1. **Vault Secrets**
- Uses Jenkins' Vault plugin to fetch API tokens for:
  - SonarQube
  - Snyk
  - JFrog
  - Jira
  - DAST/ZAP (optional)
- Assumes secrets are stored under `secret/<toolname>` paths.

### 2. **AWS STS Credentials**
- Dynamically generates short-lived credentials using Vault’s `aws/creds/<role>` endpoint.

### 3. **Source Checkout**
- Pulls the main branch from a GitHub repository.

### 4. **Security Scanning**
- **SonarQube**: Static analysis, auto-raises Jira if CRITICAL/BLOCKER issues found.
- **Snyk**: IaC scan, with JSON parsing and ticketing.
- **JFrog Xray**: Scans uploaded artifact, raises Jira for vulnerabilities.
- *(DASTardly & ZAP commented out but available.)*

### 5. **Artifact Management**
- Dummy artifact (`test.zip`) is built and pushed to JFrog.
- Scan metadata is attached via JFrog CLI for traceability.

### 6. **Fail-Fast Security Gate**
- Pipeline is aborted if any scanner detects high-risk issues.

- ### Configuring DAST Tools
This pipeline supports DASTardly and OWASP ZAP for dynamic security testing. To enable these scans:
1. **Set Up DASTardly**:
   - Deploy a DASTardly server and note its URL (replace `<your-dast-server-url>`).
   - Store credentials in Vault at `secret/dastardly` with keys `username` and `token`.
   - Set `RUN_DAST` to `true` when triggering the pipeline.
2. **Set Up ZAP**:
   - Install ZAP and expose it at `http://<your-zap-server-ip>:8080`.
   - Generate an API key and store it in Vault at `secret/zap` with key `token`.
   - Ensure `zap-cli` is installed on the Jenkins agent.
3. **Dependencies**:
   - Install `curl` (for DASTardly) and `zap-cli` (for ZAP) on the Jenkins agent.
   - Test connectivity to both servers before running the pipeline.

---

## 🛠️ Prerequisites

- Jenkins with:
  - Pipeline, Credentials, and HashiCorp Vault plugins
  - `jq`, `vault`, `curl`, `sonar-scanner`, `snyk`, `jfrog` installed on agents
- Valid Vault secrets setup and AppRole permissions
- JFrog CLI and API access set up
- Jira API access and project key

---

## 🚀 Future Enhancements

| Feature | Description |
|--------|-------------|
| **Athena Integration** | Query processed logs via AWS Athena, build dashboards or join across datasets |
| **Lambda DLQ** | Add SQS DLQ for log-processing Lambda for resilience and traceability |
| **Security Hub DLQ** | Capture and triage failed events from Security Hub via separate queue for auditing |

---

## 📁 Secrets Structure in Vault

```text
secret/
├── aws-creds
├── sonarqube
├── snyk
├── jfrog
├── jira
├── dastardly
└── zap
```

---

## 🧩 Jira Ticket Format

Tickets auto-created contain:
- Vulnerability summary
- Affected file, line, and snippet (if available)
- CVE / Severity / Fix guidance (if available)

---

## 🧪 Testing

You can test individual scanners by commenting out unrelated stages. For full pipeline simulation:
```bash
# Replace with your actual Jenkinsfile location
jenkins-jobs --conf jenkins.ini update jobs/
```

---

## 🔒 Security Notes

- All secrets are handled via Vault — no hardcoded tokens.
- AWS credentials are short-lived (STS) and purged after use.
- Only essential environment variables are exposed in the shell context.

---

## 📃 License

MIT – free to use, modify, and extend for your secure DevSecOps workflows.
