# AWS Cost Monitoring (CloudFormation)

CloudFormation template that creates a monthly and a daily AWS Budget plus a Cost Anomaly Detection monitor,
all sending email alerts to a single address.

> **Status:** legacy reference template (2026), not actively maintained. Review and update before deploying.

## Architecture

Single template: `Costs-Monitoring-Template.yaml`.

- **Monthly budget** (`<Name>-MONTLY`, COST, `AmountMonthly`): email alerts when actual spend exceeds
  `FirstThreshold`% and `SecondThreshold`%, and when forecasted spend exceeds 100%.
- **Daily budget** (`<Name>-DAILY`, COST, `AmountDaily`): email alerts when actual spend exceeds
  `FirstThreshold`% and `SecondThreshold`%.
- **Cost Anomaly Detection monitor** (`AWS::CE::AnomalyMonitor`, dimensional by `SERVICE`).
- **Three anomaly subscriptions** (`100USD`, `200USD`, `300USD`) with daily frequency and an absolute
  impact threshold of 100, 200 and 300, each emailing the same address.

## Parameters

| Name | Purpose | Default |
|------|---------|---------|
| `Name` | Budget name prefix and anomaly monitor name | `BudgetMyCompanyName` |
| `AmountMonthly` | Monthly budget limit (e.g. `2000`) | - |
| `AmountDaily` | Daily budget limit (e.g. `50`) | - |
| `Currency` | Budget currency (`USD`/`EUR`) | `USD` |
| `FirstThreshold` | First alert threshold, % of budget | `85` |
| `SecondThreshold` | Second alert threshold, % of budget | `100` |
| `Email` | Address that receives all budget and anomaly alerts | `<EMAIL>` |

## Deploy

No IAM resources are created, so no `--capabilities` flag is needed. Budgets and Cost Explorer are global
services; deploy in `us-east-1`.

```bash
aws cloudformation deploy \
  --region us-east-1 \
  --stack-name cost-monitoring \
  --template-file Costs-Monitoring-Template.yaml \
  --parameter-overrides \
      Name=<BUDGET_NAME> AmountMonthly=2000 AmountDaily=50 Email=<EMAIL>
```

AWS sends a confirmation email for the anomaly subscriptions; budget alerts go straight to the address.

## Security notes

- No security-sensitive resources. Alerts go to plain email (no SNS topic), so whoever controls that
  mailbox sees account spend information.
- The `ParameterGroups` metadata lists a parameter `Amount` that does not exist (the real ones are
  `AmountMonthly` and `AmountDaily`); this only affects the console layout.
- The anomaly subscriptions use the older `Threshold` property; newer templates use `ThresholdExpression`.

## Cleanup

```bash
aws cloudformation delete-stack --region us-east-1 --stack-name cost-monitoring
```

No manual cleanup is needed.

## License

MIT — see [LICENSE](LICENSE).
