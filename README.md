# Jenkins Secure CI/CD Pipeline with Vault, Snyk, SonarQube, and JFrog

This Jenkins pipeline automates the CI/CD process with a strong emphasis on security and quality assurance. It integrates HashiCorp Vault, SonarQube, Snyk, JFrog Artifactory/Xray, and Jira to provide a full-stack DevSecOps workflow.

## Features

- **Secrets Management** via HashiCorp Vault (AppRole-based)
- **Static Code Analysis** using SonarQube
- **Infrastructure as Code (IaC) Scanning** with Snyk
- **JFrog Artifactory** for artifact storage
- **JFrog Xray** for binary vulnerability scanning
- **Jira Integration** to automatically raise tickets for critical findings
- **Optional DAST** scanning using DASTardly and OWASP ZAP (commented out)
- **Fail-fast security gate** if critical vulnerabilities are detected

## Pipeline Stages

1. **Fetch Vault Credentials**: Retrieves static secrets from Vault for integrations (SonarQube, Jira, Snyk, etc.)
2. **Fetch AWS STS Credentials**: Generates temporary AWS credentials via Vault for secure usage.
3. **Checkout Code**: Pulls the latest code from GitHub.
4. **Static Code Analysis (SonarQube)**: Scans code for vulnerabilities and optionally creates Jira tickets for blockers/critical issues.
5. **Snyk Security Scan**: Scans IaC configurations for known security flaws.
6. **Build Artifact**: Simulates building and packaging an artifact.
7. **Upload to JFrog**: Uploads the built artifact to Artifactory.
8. **JFrog Xray Scan**: Performs vulnerability analysis on the uploaded artifact.
9. **Fail on Security Scan Issues**: Halts the pipeline if any scan reports critical issues.

> Note: DASTardly and OWASP ZAP scans are included but commented out. You can enable them as needed.

## Requirements

- Jenkins with required plugins:
  - [HashiCorp Vault Plugin](https://plugins.jenkins.io/hashicorp-vault-plugin/)
  - Pipeline Utility Steps (`readJSON`, `writeFile`, etc.)
  - Credentials Plugin
- JFrog CLI, Snyk CLI, Sonar Scanner installed on agents
- Vault configured with required secrets
- Jira project with API access
- Environment Variables set via Jenkins or vault secrets

## Setup Instructions

1. Replace placeholders like:
   - `<your-sonarqube-url>`
   - `<your-jira-site>`
   - `<your-artifactory-url>`
   - `<your-org>`, `<your-repo>`, `<your-project-key>`, etc.
2. Ensure you have Jenkins credentials set up for:
   - Vault AppRole login (`vault-role-id` and `vault-secret-id`)
   - Vault static secrets (`vault-approle`)
3. Install required CLIs (SonarQube, JFrog, Snyk) on Jenkins agent nodes.
4. Confirm `vault`, `jq`, and `curl` are available on agent nodes.

## Security Best Practices

- All tokens and sensitive data are pulled from Vault — **no hardcoded secrets**.
- AWS credentials are obtained dynamically using Vault's AWS STS backend.
- Pipeline fails early on critical issues, enforcing DevSecOps practices.

## Troubleshooting

- Check Jenkins logs for errors if a stage fails.
- Double-check Vault paths and policies if secrets aren’t retrieved.
- Ensure your Jira user has API access with the right permissions.

## Optional Enhancements

- Add Slack/MS Teams notification for scan results.
- Integrate with TruffleHog or Checkov for additional scanning.
- Replace dummy artifact build with actual build logic.

---

This project is meant to serve as a **template for secure, automated pipelines** following DevSecOps principles.

