# Robot Shop — Availability SLO and Error-Budget Policy

## 1. Purpose

Define a measurable availability objective for the Robot Shop web entry point and a policy for responding when reliability falls below the target.

## 2. Service in scope

- Service: Robot Shop web frontend
- Kubernetes namespace: `robot-shop`
- Kubernetes deployment: `web`
- Intended user journey: open the storefront and receive a successful HTTP response.

## 3. Service Level Indicator (SLI)

The preferred SLI is the proportion of valid frontend HTTP requests that return successful responses.

SLI = successful eligible requests / total eligible requests

A request is successful when it returns an HTTP status from 200 through 399. Failed requests include HTTP 5xx responses and requests that fail to complete. Exclude health checks and other synthetic traffic from the user-request SLI where practical.

For a lab environment without request-level metrics, use a synthetic HTTP probe against the web frontend as a temporary availability indicator. Document the measurement method and its limitations.

## 4. Service Level Objective (SLO)

Target: 99% successful requests over a rolling 30-day window.

This is a proposed lab objective, not a claim of measured production availability.

## 5. Error budget

For a 30-day period:

- Total time: 43,200 minutes
- SLO: 99% availability
- Error budget: 1% of the period
- Time-equivalent error budget: 432 minutes (7 hours, 12 minutes)

For request-based measurement, the error budget is 1% of eligible requests. The time equivalent is illustrative and must not replace request-based measurement when request metrics are available.

## 6. Error-budget policy

- More than 50% of the budget consumed: review recent incidents and prioritize reliability work.
- 100% of the budget consumed: pause non-essential risky releases until the service is stable and a corrective-action plan is agreed.
- Any critical security incident or sustained customer-impacting outage can trigger a release pause regardless of budget.
- Resume normal releases after service health is restored and required mitigations are verified.

## 7. Alerting policy

Create a warning when the measured success ratio falls below 99% for five consecutive minutes. Escalate sustained or severe failures according to the incident runbook.

A live alert is only considered implemented after the query, alert rule, notification route, and delivery have been tested successfully. Slack notification delivery requires a valid, securely stored Slack webhook or supported integration.

## 8. Measurement and limitations

The local Minikube cluster is a learning environment, not a production monitoring platform. Results can be affected by host resource pressure, cluster restarts, and synthetic traffic. Record the measurement source, observation window, and test conditions with any reported result.

Do not claim that the target was achieved unless monitoring evidence supports it.

## 9. Review

Review this objective after the lab exercise. In a real service, set the SLO using user expectations, business impact, and observed reliability data.
