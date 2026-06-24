# Ecommerce-CID-Agent
This is an ecommerce checkout incident detection agent, that consistently reviews new events, decides whether the situation warrants attention, and publish a clean “command” for downstream execution. 

**Problem Statement**
E-commerce teams monitor checkout activity to catch issues that customers face during an order checkout process whcih can impact revenue(especially payment failures). At scale, in real production scenarios hundreds of checkout events may arrive every hour and it’s not practical to manually triage each one of them. 

**Goal**
Is to build an E-commerce Agent in Zapier that reads new checkout events from Google sheets, evaluates their severity using baseline metrics, and outputs exactly one command per run (ALERT or LOG). The agent does not execute fixes; it only publishes intent via a webhook and updates state. 

**Requirements**
1. **State Management (Critical)** 
    a. The agent must read the last_processed_timestamp from the state sheet at the start of every run. 
    b. It must filter the cart_events to only process rows where the timestamp is newer than the stored state and corresponding processed column is empty.
    c. At the end of the run, it must update the state sheet with the timestamp of the latest processed event.
    d. Maintain the last CMD ID for next iteration
   <img width="792" height="811" alt="image" src="https://github.com/user-attachments/assets/66f4245a-1084-4832-841c-5ad29076f176" />

2. **Decision Logic**
    The agent must evaluate every new event based on these business rules: 
    a. Severity High: If the issue occurs during the PAYMENT step (critical revenue impact). 
    b. Severity High: If the cart_value is significantly high. 
    c. Severity High: If the failure rate exceeds the baseline_rate. 
    d. Severity High: If multiple mmobile failures occur (potential app bug). 
    e. Action: 
        ALERT: If severity is HIGH. 
        LOG: If severity is LOW (e.g., low value, isolated incident).
  <img width="1848" height="655" alt="Screenshot 2026-06-17 145950" src="https://github.com/user-attachments/assets/2e1c2821-202d-4490-8f72-7c4b6a048a0f" />


3. **Execution & Hand-off**
    a. The agent must write exactly one row to the Agent_Output sheet per decision. 
    b. Mandatory: The agent must trigger a Webhook (POST request) to broadcast its decision to downstream systems. 
    c. The Webhook JSON body must contain all 9 fields from the command row (action, severity, reason, etc.).
   data/Webhook Endpoint.png

Additional Requirements 
1. Zero Duplicates: The agent never processes the same checkout event row twice (State Management works). To Handle this included a Processed Column , to be marked "Yes" each time agent has read and processed the row. 
2. Smart Triage: The agent correctly identifies high-value Payment failures as HIGH/ALERT
3. Human Review Feasibility: Store the CMDs generated for every run inorder to audit or review later on, this gives more debugging capability.
4. Connectivity: The Webhook fires successfully with a valid JSON payload for every decision made.

**System Design**
assets/CID_Agent.png

**Scaling Strategy**
docs/Scaling_Strategy.md
