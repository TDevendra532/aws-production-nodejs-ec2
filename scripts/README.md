# Scripts

Helper scripts for the project (planned):

- `bootstrap.sh` — initial setup (creates tfstate bucket + DynamoDB lock table)
- `validate.sh` — runs `terraform fmt`, `tflint`, `tfsec` across all modules
- `plan-all.sh` — wraps `terraform plan` across staging and production envs
- `slack-notify.py` — Lambda subscriber for SNS-to-Slack alarm formatting
