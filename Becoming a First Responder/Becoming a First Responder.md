## TryHackMe Write-up: Becoming a First Responder

**Room:** Becoming a First Responder

**Platform:** TryHackMe

**Category:** Incident Response / Incident Management / Blue Team

**Difficulty:** Beginner-friendly

**Room Link:** [https://tryhackme.com/room/becomingafirstresponder](https://tryhackme.com/room/becomingafirstresponder)

---

### Overview

This room introduces the role of a **First Responder** during a cyber incident. A First Responder is not always a SOC analyst or a dedicated incident responder. Sometimes, the first person to notice an incident is a security engineer, system owner, developer, product team member, or another employee responsible for a system.

The main idea of the room is that the First Responder does not need to solve the whole incident alone. Their priority is to avoid making the situation worse, preserve evidence, notify the correct people, support containment, and document everything clearly until the Blue Team or Incident Response team takes over.

The room focuses on the following concepts:

* Preserving evidence correctly
* Understanding the volatility of evidence
* Avoiding common mistakes during an incident
* Maintaining chain of custody
* Using playbooks and call trees
* Understanding containment methods
* Knowing when Business Continuity Planning is required
* Documenting incident actions in a reliable format

---

### Task 2: Evidence Preservation

### Key Concept

The first major responsibility of a First Responder is to **preserve evidence**. When a system is compromised, it may contain important information that can help investigators understand what happened, how the attacker entered, what actions were performed, and whether other systems were affected.

A very common mistake during incidents is turning off the compromised machine. This is dangerous because many important artefacts are volatile and disappear when the system loses power.

Examples of volatile evidence include:

* Running processes
* Active network connections
* RAM contents
* ARP cache
* Routing table
* Temporary files
* Malware payloads loaded only in memory

Another reason not to immediately shut down or disconnect a host is that it may alert the attacker. If the attacker realizes that defenders are investigating, they may delete evidence, escalate the attack, deploy ransomware, or move to other systems.

### Order of Volatility

Evidence should be preserved according to how quickly it can disappear. The room follows this priority order:

| Priority | Evidence Type                                                         | Explanation                                                                                     |
| -------: | --------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
|        1 | Registers and Cache                                                   | Extremely volatile; changes almost instantly during execution.                                  |
|        2 | Routing Table, ARP Cache, Process Table, Kernel Statistics and Memory | Useful for understanding active communication, running processes, and malware behaviour in RAM. |
|        3 | Temporary File Systems                                                | May contain active session data, temporary payloads, or temporary application files.            |
|        4 | Disk                                                                  | Contains files, local logs, malware samples, persistence mechanisms, and user data.             |
|        5 | Remote Logging and Monitoring                                         | SIEM, EDR, and centralised logs that may have retention limits.                                 |
|        6 | Physical Configuration and Network Topology                           | Helps understand where the host sits in the network and what other systems may be affected.     |
|        7 | Archival Media                                                        | Backups and archived data; usually less volatile but useful for historical comparison.          |

### Important DON'Ts

The room highlights three major mistakes to avoid:

1. **Do not shut down the compromised device.**
   This destroys volatile evidence and may alert the attacker.

2. **Do not blindly trust programs on the compromised system.**
   Attackers may have modified system tools, so using them could produce unreliable results or taint evidence.

3. **Do not run programs that modify file access times.**
   Even simple actions like copying files can change metadata and damage the evidentiary value of the artefacts.

### Chain of Custody

For evidence to be useful in legal proceedings, it must be possible to prove that it was not tampered with. This process is known as **Chain of Custody**.

Chain of Custody documents:

* What evidence was collected
* Who collected it
* When it was collected
* Where it was stored
* Who accessed it
* What actions were performed on it
* Whether analysis was performed on a copy rather than the original

This helps prove that the evidence remained reliable and admissible.

### Task 2 Answers

| Question                                                                                   | Answer             |
| ------------------------------------------------------------------------------------------ | ------------------ |
| What priority order for preservation is given for the Disk?                                | `4`                |
| What priority order for preservation is given for Archival Media?                          | `7`                |
| What priority order for preservation is given for the Register and Cache?                  | `1`                |
| What is the term used to describe ensuring that evidence can be used in legal proceedings? | `Chain of Custody` |

---

### Task 3: Alerting the Right People

### Key Concept

After evidence preservation has started, the next responsibility of the First Responder is to notify the correct stakeholders. A First Responder should not try to improvise the full response alone. Instead, they should follow the organisation’s predefined process.

Two important concepts are introduced here:

* **Playbooks**
* **Call Trees**

### Playbooks

A **playbook** is a predefined process that explains what steps should be followed during a specific type of incident.

For example, an organisation may have separate playbooks for:

* Phishing incidents
* Malware infections
* Account compromise
* Data leakage
* Ransomware
* Insider threats

The purpose of a playbook is to make the response repeatable, structured, and less dependent on improvisation. During a stressful incident, a clear playbook helps the team avoid missing important steps.

A playbook may include:

* Initial triage steps
* Required evidence to collect
* Stakeholders to notify
* Containment actions
* Escalation paths
* Recovery steps
* Documentation requirements

### Call Trees

A **call tree** defines who must be informed during an incident and in which order.

In larger organisations, incidents may be raised through ticketing systems such as Jira or ServiceNow. However, call trees are still important because they clearly define responsibility and escalation.

A call tree usually answers questions like:

* Who should be contacted first?
* Who is responsible for notifying management?
* Who should be contacted if the first person is unavailable?
* When should the Blue Team be involved?
* When should senior management or crisis management be involved?

### Responsibility of the First Responder

The First Responder may not be responsible for writing full SOC playbooks, but they should know:

* How to report an incident
* Who to contact
* Which escalation path to follow
* What information to provide
* How to avoid damaging evidence before handover

### Task 3 Answers

| Question                                                                                                  | Answer      |
| --------------------------------------------------------------------------------------------------------- | ----------- |
| What is the term that describes a defined process that the blue team follows during an incident?          | `Playbook`  |
| What is the term that describes the structure used to inform all the relevant parties about the incident? | `Call Tree` |

---

### Task 4: Containment

### Key Concept

Once the correct people have been notified, the next major phase is **containment**. Containment means stopping the incident from spreading or causing more damage.

The room explains that containment, eradication, and recovery must happen in the correct order:

```text
Containment → Eradication → Recovery
```

This order is important. If defenders attempt eradication or recovery before containment, the attacker may still have access and can simply compromise the environment again.

### Containment vs Eradication vs Recovery

| Phase       | Meaning                          | Example                                                            |
| ----------- | -------------------------------- | ------------------------------------------------------------------ |
| Containment | Stop the incident from spreading | Isolate an infected host from the network                          |
| Eradication | Remove the threat                | Delete malware, disable persistence, reset compromised credentials |
| Recovery    | Restore normal operations        | Rebuild systems, restore services, return to Business as Usual     |

### Containment Methods

The room explains three main containment methods.

#### 1. Network Segmentation

The infected host is isolated from the rest of the network by moving it to a different network segment or restricting its communication.

Goal:

* Prevent communication with other internal hosts
* Limit lateral movement
* Reduce the spread of infection

#### 2. Physical Isolation

The infected host is physically collected or confiscated by the Blue Team.

Goal:

* Prevent further user or attacker interaction
* Preserve evidence
* Ensure the system is fully isolated from the network

#### 3. Virtual Isolation

The host is isolated remotely using software, often through an EDR platform. This is sometimes called “jailing” the host.

Goal:

* Restrict communication remotely
* Allow the Blue Team to contain the system without physically accessing it
* Maintain some controlled communication for investigation if needed

### Rate Limiting

The room also mentions a useful defensive technique: slowing down the attacker instead of fully cutting access immediately.

In some cases, defenders may rate-limit the host’s connection to gain time for investigation. This can make the attacker’s command-and-control communication unreliable without clearly revealing that defenders are aware of the compromise.

### Task 4 Answers

| Question                                                                                           | Answer                 |
| -------------------------------------------------------------------------------------------------- | ---------------------- |
| What containment method can be performed remotely using the EDR?                                   | `Virtual Isolation`    |
| What containment method requires the blue team to collect the infected host?                       | `Physical Isolation`   |
| What containment method aims to ensure that the infected host cannot communicate with other hosts? | `Network Segmentation` |

---

### Task 5: Business Continuity Planning

### Key Concept

If an incident is severe enough, the organisation may need to invoke its **Business Continuity Plan**, also called **BCP**.

A BCP is designed to help an organisation continue operating or recover operations during and after a serious incident.

However, the room makes one point very clear: BCP should not be invoked before containment. If recovery starts while the attacker still has access, the recovered systems may simply be compromised again.

### What is BCP?

**BCP** stands for **Business Continuity Plan**.

It is a broad plan that helps the organisation continue operating during disruption. It can include:

* Communication with internal stakeholders
* Communication with external stakeholders
* Recovery planning
* Business priorities
* Alternative operating procedures
* Crisis coordination
* Technical recovery steps

### What is DRP?

**DRP** stands for **Disaster Recovery Plan**.

A DRP focuses more specifically on the technical recovery of systems, infrastructure, data, and services.

A DRP is usually part of the wider BCP.

### BCP vs DRP

| Plan | Full Name                | Focus                                            |
| ---- | ------------------------ | ------------------------------------------------ |
| BCP  | Business Continuity Plan | Keeping the business operating during disruption |
| DRP  | Disaster Recovery Plan   | Technical recovery of systems and services       |

### Creating a BCP

The room describes four main steps for creating an effective BCP:

1. **Perform a Business Impact Analysis**
   Understand how an incident could affect the organisation, users, customers, and services.

2. **Define Potential Recovery Actions**
   Identify practical actions such as switching to disaster recovery systems or restoring from backups.

3. **Plan the BCP Team Structure**
   Define who is responsible for communication, documentation, approval, and execution.

4. **Test the BCP Plan**
   Use tabletop exercises and training to verify that the plan works.

### BCP Metrics

The room introduces several important BCP metrics.

| Metric | Full Name                  | Meaning                                                 |
| ------ | -------------------------- | ------------------------------------------------------- |
| RPO    | Recovery Point Objective   | Maximum amount of data loss the organisation can accept |
| RTO    | Recovery Time Objective    | Time required to recover the hardware or system         |
| WRT    | Work Recovery Time         | Time required to recover software and data              |
| MTD    | Maximum Tolerable Downtime | Maximum acceptable downtime                             |
| MTBF   | Mean Time Between Failures | Average time between failures or incidents              |
| MTTR   | Mean Time To Repair        | Average time required to repair or recover the system   |

### Task 5 Answers

| Question                                                                                               | Answer                     |
| ------------------------------------------------------------------------------------------------------ | -------------------------- |
| What does BCP stand for?                                                                               | `Business Continuity Plan` |
| What does DRP stand for?                                                                               | `Disaster Recovery Plan`   |
| What BCP metric is used to describe the amount of time required to recover the hardware of our system? | `Recovery Time Objective`  |
| What BCP metric is used to describe the average amount of time required to recover our system?         | `Mean Time To Repair`      |

---

### Task 6: Documentation

### Key Concept

Documentation is critical during an incident, especially when BCP is invoked. When an organisation activates BCP, some normal change management steps may be bypassed to respond faster. This makes documentation even more important because the team must still be able to reconstruct what happened.

Good incident documentation helps with:

* Handover to the Blue Team
* Timeline reconstruction
* Accountability
* Legal review
* Lessons learned
* Post-incident improvement

### What Should Be Documented?

The room recommends documenting the following information:

| Documentation Field                          | Purpose                                          |
| -------------------------------------------- | ------------------------------------------------ |
| Time the action was requested                | Helps build the incident timeline                |
| Description of the action                    | Shows what was planned or performed              |
| Reason for the action                        | Explains the decision-making process             |
| Person approving the action                  | Provides accountability                          |
| Person responsible for performing the action | Prevents confusion about ownership               |
| Time the action was performed                | Allows comparison with logs and alerts           |
| Observed changes after the action            | Shows whether the action had the intended effect |

### Why UTC Matters

Incident notes should use **UTC** as the standard time format. This avoids confusion when different systems, teams, or logs use different time zones.

Using UTC makes it easier to correlate:

* SIEM logs
* EDR alerts
* Firewall logs
* Email gateway logs
* Cloud logs
* Analyst notes
* Timeline events

### Lessons Learned

Documentation is also useful after the incident is closed. During the post-incident review, the team can analyse what happened, what worked, what failed, and what should be improved.

This process helps reduce future incidents and improves the organisation’s incident response maturity.

### Task 6 Answer

| Question                                                                              | Answer |
| ------------------------------------------------------------------------------------- | ------ |
| What time format should be used in our incident notes to ensure that all times match? | `UTC`  |

---

### Final Notes

This room is useful because it teaches that a First Responder’s job is not to “fix everything immediately”. The priority is to act carefully, avoid destroying evidence, notify the right people, support containment, and document everything.

The most important lessons are:

* Do not shut down a compromised host immediately.
* Preserve evidence in order of volatility.
* Maintain Chain of Custody for legal reliability.
* Use playbooks and call trees instead of improvising.
* Containment must happen before eradication and recovery.
* BCP helps restore business operations, while DRP focuses on technical recovery.
* Incident documentation should use UTC.
* Every action should have a reason, an owner, an approver, and a timestamp.

---

### Quick Reference

| Concept              | Meaning                                                |
| -------------------- | ------------------------------------------------------ |
| First Responder      | The first person who identifies or reports an incident |
| Chain of Custody     | Process used to prove evidence was not tampered with   |
| Playbook             | Defined process followed during an incident            |
| Call Tree            | Structure used to notify relevant stakeholders         |
| Containment          | Stopping the incident from spreading                   |
| Virtual Isolation    | Remote isolation using EDR                             |
| Physical Isolation   | Physically collecting and isolating the host           |
| Network Segmentation | Isolating the host at the network level                |
| BCP                  | Business Continuity Plan                               |
| DRP                  | Disaster Recovery Plan                                 |
| RTO                  | Time required to recover hardware/system               |
| MTTR                 | Average time required to repair/recover                |
| UTC                  | Standard time format for incident notes                |

---


### Personal Reflection

This room helped me understand that the first minutes of an incident are extremely important. A rushed action, such as shutting down a machine or copying files incorrectly, can destroy valuable evidence. A good First Responder should stay calm, preserve the scene, follow predefined processes, and communicate clearly with the correct teams.

The main takeaway is that incident response is not only technical. It also depends on preparation, communication, documentation, responsibility, and disciplined decision-making.
