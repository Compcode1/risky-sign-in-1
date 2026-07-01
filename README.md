# Microsoft Entra ID Protection: Real-Time Risk-Based Session Enforcement

## Project Overview
This project demonstrates the deployment, isolation, and verification of automated session-blocking controls using real-time threat intelligence within Microsoft Entra ID (MEID). By leveraging Microsoft Entra ID Protection (MEIDP) signals, we established an inline enforcement gate that evaluates identity risk during the authentication handshake. The primary objective was to intercept anomalous access attempts—specifically anonymous proxy traffic—and terminate the authentication pipeline before an adversarial session could issue an Access Token (AT) or Refresh Token (RT).

---

## Architecture & Policy Configuration

The implementation utilizes a dedicated risk-based Conditional Access (CA) policy targeted globally across the directory, while maintaining strict boundary isolation to protect core administration paths during testing.

### Policy Blueprint: `ca-sign-in-risk-block`

| Configuration Field | Applied System Parameter | Architectural Impact |
| :--- | :--- | :--- |
| **Users (Include)** | All Users | Blanket coverage ensuring all standard and administrative identities are monitored globally. |
| **Users (Exclude)** | Primary Global Administrator Account | Hard boundary exclusion to prevent accidental administrative lockout during live threat testing. |
| **Target Resources** | All Cloud Apps / Resources | Extends the risk evaluation gate to every data container and endpoint connected to the tenant. |
| **Conditions** | Sign-in Risk: **Medium** and **High** | Triggers the policy engine when real-time telemetry (e.g., anonymous IP, atypical travel) crosses moderate threat thresholds. |
| **Access Controls** | Grant -> **Block Access** | Intercepts the handshake and rejects token issuance instantly upon a risk match. |
| **Enable Policy** | **On** | Writes the policy directly to the active directory runtime schema. |

---

## Live Simulation & Testing Methodology

To validate the inline enforcement gate, an anomalous sign-in event was simulated from outside the organization's standard operational baselines. 

1. **Tooling:** A client session was initiated using a Tor-routed browser instance to simulate egress traffic originating from an open, anonymous proxy network node.
2. **Target Destination:** The browser navigated directly to the centralized application gateway (`https://myapps.microsoft.com`).
3. **Identity Selection:** Authentication was attempted using a standard targeted directory account (`important one`).

### Client-Side Results
The moment credentials were submitted, Microsoft Entra ID intercepted the request. Because the identity engine flagged the traffic source as an anonymous proxy, the session was blocked prior to dashboard rendering. The browser displayed the explicit authorization error page:
> *"You cannot access this right now. Your sign-in was successful, but does not meet the criteria to access this resource."*

---

## Telemetry Findings & Audit Log Analysis

After pipeline ingestion latency concluded, the security event successfully populated the administrative monitoring ledger under **Microsoft Entra ID** > **Protection** > **Identity Protection** > **Risky sign-ins**. 

The authoritative logging engine captured the following telemetry payload:

* **Target Principal:** `important one`
* **Risk State:** `At risk`
* **Inferred Telemetry Location:** Delaware, USA *(Reflecting the physical registration of the anonymous Tor network exit node proxy)*
* **Trigger Mechanism:** Real-time machine learning detection of an unverified/anonymous IP routing layer.
* **Enforcement Outcome:** Hard block executed successfully. The log tab confirms that the `ca-sign-in-risk-block` Conditional Access policy successfully evaluated the inbound claims and suppressed the creation of session tokens.

## Project Conclusion
The architecture successfully passed validation. The real-time evaluation layer functions as designed, reliably identifying infrastructure anomalies and dropping a definitive access gate to prevent credential-stuffing or session-hijacking plays from anonymized networks.
