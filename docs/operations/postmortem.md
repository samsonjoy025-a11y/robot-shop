# Robot Shop - Blameless Incident Postmortem

**Status:** Lab incident drill completed; documentation recorded retrospectively  
**Environment:** Local Minikube lab  
**Namespace:** `robot-shop`  
**Affected service:** Robot Shop web frontend

## 1. Incident summary

A controlled availability drill was performed by scaling the `web` deployment to zero replicas and then restoring it to one replica.

- **Incident type:** Planned, simulated availability outage
- **Start time:** Record from terminal history if available
- **Recovery time:** Record from terminal history if available
- **Severity:** Lab simulation; no production impact
- **Affected component:** `deployment/web`
- **Data stores:** No database or other application deployments were intentionally changed.

## 2. Impact

The web deployment was intentionally scaled to zero replicas. Kubernetes reported `0/0` replicas available, meaning the frontend had no web pods available to serve requests during the simulated outage.

The other Robot Shop application pods were shown as Running during the drill. No actual user traffic or customer impact was measured.

## 3. Timeline

1. **Pre-drill:** The web deployment was healthy with `1/1` available replica.
2. **Simulation:** `kubectl scale deployment/web -n robot-shop --replicas=0` succeeded.
3. **Outage confirmation:** Kubernetes reported the web deployment at `0/0` available replicas.
4. **Recovery attempt:** The first scale request to restore one replica timed out at the API request level.
5. **State check:** A subsequent deployment query confirmed the web deployment remained at zero replicas.
6. **Recovery:** `kubectl scale deployment web -n robot-shop --replicas=1 --request-timeout=60s` succeeded.
7. **Recovery verification:** Kubernetes reported `1/1` available replica.

Exact timestamps and total duration were not recorded in this summary. Add them only if you can verify them from terminal history.

## 4. Root cause and contributing factors

- **Immediate cause of the simulated outage:** The web deployment was deliberately scaled to zero replicas as part of the drill.
- **Recovery complication:** A Kubernetes API request timed out during the first recovery attempt.
- **Underlying cause of timeout:** Not determined. The timeout alone does not establish why the API request failed.

## 5. Response and recovery

The deployment state was checked after the timeout rather than assuming the request had succeeded. The scale command was then retried with a longer request timeout.

Recovery was confirmed when Kubernetes reported:

- Desired replicas: 1
- Ready replicas: 1
- Available replicas: 1

No database deletion or persistent-data modification was performed during the drill.

## 6. What went well

- The deployment state was checked before and after the drill.
- The recovery request was retried only after confirming the deployment was still at zero replicas.
- The web deployment returned to its original available replica count.

## 7. What could be improved

- Record exact incident and recovery timestamps during future drills.
- Investigate intermittent Kubernetes API timeouts if they recur.
- Ensure monitoring and alert delivery are tested separately; this drill did not verify that an automated alert fired.
- Verify the storefront through an HTTP request after recovery.

## 8. Corrective actions

| Action | Priority | Status |
|---|---|---|
| Verify web frontend response after recovery | Medium | Pending verification |
| Investigate repeated Kubernetes API timeouts if observed again | Medium | Monitor |
| Complete Loki and Fluent Bit log collection | Medium | Pending |
| Verify a live availability alert and notification route | Medium | Pending |

## 9. Blameless learning

This planned drill demonstrated the importance of checking the actual Kubernetes deployment state after a command times out. The timeout should be investigated as a system reliability issue if it recurs; no individual blame is assigned.

## 10. Closure

The web deployment was restored to `1/1` available replica. The incident drill is complete. Final HTTP availability verification and automated alert verification remain separate tasks.

