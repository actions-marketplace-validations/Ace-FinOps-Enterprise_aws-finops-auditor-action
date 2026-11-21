# AWS FinOps Cost Auditor (GitHub Action)

A zero-agent, read-only GitHub Action powered by [`aws-finops-auditor`](https://pypi.org/project/aws-finops-auditor/) to detect orphaned EBS volumes, idle EC2 instances, and unattached Elastic IPs directly in CI/CD workflows.

## Quickstart

```yaml
name: AWS FinOps Cost Audit

on:
  pull_request:
  schedule:
    - cron: '0 8 * * 1'

permissions:
  id-token: write
  contents: read

jobs:
  finops-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::<YOUR_AWS_ACCOUNT_ID>:role/GitHubActionsFinOpsRole
          aws-region: us-east-1

      - name: Run AWS FinOps Cost Audit
        id: finops
        uses: Ace-FinOps-Enterprise/aws-finops-auditor-action@v1
        with:
          fail-on-waste: '100'
          gumroad_license_key: ${{ secrets.GUMROAD_LICENSE_KEY }}
```

> `gumroad_license_key` is optional. Without it the Action runs free: it reports waste totals and exits 0. With a Pro key it also generates Bash remediation artifacts, CSV compliance exports, and enforces `fail-on-waste`.

> Without `file`, the Action runs a live scan against your AWS account using the credentials from the Configure AWS Credentials step (`region` defaults to `us-east-1`; set it to scan elsewhere). Pass `file` for an offline CSV audit instead. Live scanning is a free core capability: it runs the same read-only checks and never triggers the license check.

## Inputs & Outputs

| Input | Description | Default |
| :--- | :--- | :--- |
| `file` | Optional CSV path for offline/demo audits | `''` |
| `region` | AWS region(s) to scan in live mode, comma-separated | `us-east-1` |
| `fail-on-waste` | Fail workflow if monthly USD waste exceeds threshold **(Pro)** | `''` |
| `gumroad_license_key` | Pro subscription key, passed as `${{ secrets.GUMROAD_LICENSE_KEY }}` | `''` |
| `package-version` | PyPI version constraint for `aws-finops-auditor` | `>=1.2.0` |

| Output | Description |
| :--- | :--- |
| `monthly-waste` | Identified monthly AWS waste in USD |
| `annual-waste` | Identified annualized AWS waste in USD |

## Pro Features

The following are **Pro** features, unlocked by passing `gumroad_license_key` (a Cloud FinOps Auditor Pro subscription, $19/month or $228/year):

- **Fail-on-waste threshold enforcement** — the workflow fails when monthly waste exceeds `fail-on-waste`. Free runs still report the waste totals and exit 0; they never fail the build on waste.
- **Bash remediation artifacts** — generates `remediate_anomalies.sh` with the cleanup commands for the waste found.
- **CSV compliance exports** — generates `compliance_audit_export.csv` for SOC2/ISO27001 audit evidence.

Get a Pro key at [Cloud FinOps Auditor Pro](https://aisser.gumroad.com/l/awmyxz) and store it as a repository secret named `GUMROAD_LICENSE_KEY`.

## Enterprise & Multi-Account Upgrades

Need automated **AWS Organizations multi-account scanning**, **executive PDF showback reports**, and **idempotent bash remediation scripts**?

- **[AWS Cloud Cost Auditor (Practitioner & Agency Tiers)](https://aisser.gumroad.com/l/krgac)**
