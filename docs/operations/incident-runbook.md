\# Robot Shop — Incident Response Runbook



\## 1. Purpose



Provide a repeatable process for investigating and recovering from a Robot Shop availability incident in the `robot-shop` Kubernetes namespace.



\## 2. Example incident



The Robot Shop storefront is unavailable, returns errors, or responds too slowly.



\## 3. Detection



Check monitoring alerts first. If monitoring is unavailable, inspect Kubernetes health.



```powershell

kubectl get pods -n robot-shop

kubectl get deployments -n robot-shop

kubectl get events -n robot-shop --sort-by=.lastTimestamp

```



\## 4. Initial triage



1\. Record the incident start time and observed symptoms.

2\. Determine whether the issue affects the web frontend or multiple services.

3\. Check pod readiness, restart counts, and recent events.

4\. Inspect the affected workload's logs.



```powershell

kubectl logs deployment/web -n robot-shop --tail=100

kubectl describe deployment web -n robot-shop

```



If the deployment name differs, identify it with `kubectl get deployments -n robot-shop`.



\## 5. Mitigation



\- Inspect pod events and logs before changing an unhealthy workload.

\- Restore the intended replica count if a deployment has no available replicas.

\- If a recent change caused the incident, identify the affected release before considering rollback.

\- Never delete databases or persistent data as a routine recovery action.

\- Avoid making several unrelated changes at once.



Inspect Helm history before considering a rollback:



```powershell

helm history robot-shop -n robot-shop

```



Use a known-good revision only when justified, then verify recovery.



\## 6. Recovery verification



```powershell

kubectl get pods -n robot-shop

kubectl get deployments -n robot-shop

kubectl rollout status deployment/web -n robot-shop --timeout=120s

```



Test the storefront through its configured service or port-forward. Confirm successful responses and recovery of relevant monitoring signals.



\## 7. Communication and escalation



Record the incident owner, start and end times, affected service, observed impact, evidence, mitigation, recovery steps, and outstanding risks.



Escalate if the service cannot be restored safely, data integrity is at risk, or a security breach is suspected.



\## 8. Closure



Close the incident only after service health is verified. Create a blameless postmortem for a significant or intentionally simulated incident. Focus on system conditions and improvements rather than individual blame.



\## 9. Lab limitations



This runbook documents the response procedure. Commands and recovery actions must be tested in the actual lab before being reported as successfully exercised.

