Here is a consolidated and structured response for your PSC + Shared VPC + Kafka + Pega scenario.

⸻

1️⃣ When is a Service Project Required?

✅ Service Project is Required When:
	•	Your application runs inside GCP (VM, GKE, Cloud Run, etc.)
	•	You are using Shared VPC architecture
	•	The workload needs to access a PSC endpoint

In this case:
	•	Network resources (VPC, subnets, PSC endpoint) live in the host project
	•	Workloads live in the service project
	•	The service project attaches to the host VPC

⸻

❌ Service Project is NOT Required When:
	•	Kafka is fully on-prem
	•	Traffic comes via Cloud VPN or Interconnect
	•	No GCP compute workload needs PSC
	•	Everything is handled from on-prem through hybrid connectivity

In that case, only the host project with PSC endpoint is enough.

⸻

2️⃣ How GCP Workload Connects to PSC

Important clarification:

A workload does NOT connect to the service attachment directly.

Flow is:
	1.	Vendor (Pega) exposes a Service Attachment
	2.	You create a PSC Endpoint (Forwarding Rule) in your host project
	3.	PSC endpoint gets an internal IP address
	4.	GCP workload connects to that internal PSC IP

So workload connects to:

✔️ PSC endpoint internal IP
❌ Not the service attachment
❌ Not vendor load balancer directly

If using Shared VPC:
	•	Workload in service project uses host VPC network
	•	Routing happens automatically
	•	Firewall rules must allow traffic

⸻

3️⃣ Can We Use DNS for PSC?

Yes — and it is recommended in most production setups.

Use DNS When:
	•	You want abstraction from IP changes
	•	Vendor requires FQDN-based access
	•	You want clean application configuration
	•	You may recreate PSC endpoint in future

Typical setup:
	•	Create private DNS zone
	•	Map FQDN → PSC internal IP
	•	Workloads connect using FQDN

Example:
kafka.vendor.internal → 10.x.x.x (PSC endpoint IP)

If static IP and simple setup:
	•	Direct IP usage is also fine

⸻

4️⃣ Observability Responsibility Split

You summarized correctly. Here is the clean ownership model:

🔹 GCP Networking Team Responsible For:
	•	VPC Flow Logs
	•	Firewall logging
	•	Load balancer (PSC forwarding rule) metrics
	•	Connectivity tests
	•	DNS logging (if private DNS used)
	•	Monitoring packet drops / unusual traffic

GCP handles infrastructure observability.

⸻

🔹 On-Prem Kafka Team Responsible For:
	•	Broker health
	•	Connection stability
	•	Producer/consumer errors
	•	Throughput and latency
	•	TLS errors
	•	Consumer lag

PSC is Layer 4 networking only — it does NOT monitor Kafka protocol health.

⸻

5️⃣ Final Clear Architecture Summary

If Kafka is On-Prem:

On-prem → VPN/Interconnect → PSC endpoint (host project) → Pega
No service project required.

If GCP Workload Consumes PSC:

Service Project Workload → Host VPC → PSC Endpoint IP → Vendor Service Attachment → Pega
Service project required.

⸻

🎯 Clean One-Line Summary
	•	Service project needed only if workload runs in GCP.
	•	Workload connects to PSC endpoint IP, not service attachment.
	•	DNS recommended for production flexibility.
	•	GCP networking monitors infrastructure.
	•	Kafka team monitors application layer.

⸻

If you want, I can also give you a simple architecture diagram explanation in text format for documentation purposes.

For Private Service Connect (PSC) you don’t monitor “PSC” as a single object — you monitor the layers around it:
	•	Network
	•	Load balancer (forwarding rule)
	•	Backend health
	•	Connectivity
	•	DNS (if used)

Here’s a structured observability checklist for PSC in GCP.

⸻

🔎 1️⃣ Network-Level Observability (Mandatory)

✅ VPC Flow Logs

Enable on the subnet where:
	•	PSC endpoint is created
	•	Service project workloads are running

This helps you see:
	•	Source IP
	•	Destination PSC IP
	•	Accepted / Denied
	•	Bytes transferred

Useful for:
	•	Debugging drops
	•	Identifying unexpected access (data leak check)

⸻

✅ Firewall Rule Logging

Enable logging on firewall rules allowing traffic to PSC endpoint.

Helps detect:
	•	Misconfigured rules
	•	Unauthorized attempts

⸻

📊 2️⃣ Load Balancer / PSC Endpoint Metrics

PSC endpoint is implemented using a forwarding rule.

Monitor in:
	•	Cloud Monitoring → Load Balancing metrics

Important metrics:
	•	forwarding_rule/bytes_sent
	•	forwarding_rule/bytes_received
	•	forwarding_rule/packets_dropped

Set alerts for:
	•	Sudden spike in traffic
	•	Unexpected traffic patterns
	•	Zero traffic (possible outage)

⸻

🩺 3️⃣ Backend Health (Critical)

If the producer side (Pega/vendor) exposes through:
	•	Internal TCP Proxy
	•	Internal HTTP(S) LB

Monitor:
	•	Backend service health status
	•	Health check failures
	•	Connection reset rates

If backend becomes unhealthy, PSC traffic will fail.

⸻

🌍 4️⃣ Connectivity Monitoring (Highly Recommended)

Use:

🔹 Connectivity Tests (Network Intelligence Center)

Create a test from:
	•	Service project VM/GKE Pod
To:
	•	PSC endpoint IP

This verifies:
	•	Routing
	•	Firewall
	•	Reachability

⸻

🧭 5️⃣ DNS Observability (If Using FQDN)

If PSC uses private DNS:

Monitor:
	•	Cloud DNS query logs
	•	NXDOMAIN errors
	•	Latency

Misconfigured DNS is a common PSC issue.

⸻

🔐 6️⃣ Security Observability (For Data Leak Concerns)

Enable:
	•	VPC Flow Logs with aggregation = fine
	•	Log-based metrics for unexpected source IPs
	•	Alert if traffic originates from unknown subnet/project

You can create alert:

If PSC endpoint receives traffic from non-approved subnet → Alert

⸻

📈 7️⃣ Application-Level Observability (Kafka Case)

Since you’re using Kafka:

Monitor:
	•	Connection timeouts
	•	Broker disconnections
	•	Latency spikes
	•	Retries
	•	Consumer lag

Because PSC only handles network — app failures won’t show in PSC metrics.

⸻

🚨 Recommended Alerts

Set alerts for:
	•	High packet drops
	•	No traffic for X minutes (unexpected outage)
	•	High connection resets
	•	Firewall denies
	•	Backend unhealthy

⸻

🎯 Minimal Required Observability (Production)

If you want a lean setup:
	1.	VPC Flow Logs (enabled)
	2.	Firewall logging (enabled)
	3.	Load balancer metrics alerts
	4.	Connectivity test configured
	5.	Application metrics monitored

⸻

🧠 Important Reality

PSC itself does NOT provide:
	•	Deep request logs
	•	Payload visibility
	•	TLS inspection

It is Layer 4 forwarding.

So most observability must come from:
	•	Network logs
	•	Load balancer metrics
	•	Application logs

⸻

If you tell me:
	•	Is this production?
	•	Is Kafka high throughput?
	•	Is data sensitivity high (financial/PII)?

I can suggest an enterprise-grade monitoring blueprint.
In a Shared VPC setup, your service project workload does NOT connect directly to the service attachment.

It connects to the Private Service Connect (PSC) endpoint IP (which is an internal load balancer IP created in the host project).

Let me clarify clearly 👇

⸻

🔹 How PSC Works in Shared VPC

1️⃣ Vendor Side (Pega)

They expose a Service Attachment (producer side).

2️⃣ Your Host Project

You create:
	•	A PSC endpoint
	•	Which gets an internal IP address (from a subnet in host project)
	•	This endpoint connects to the vendor’s service attachment

3️⃣ Service Project Workload

Your VM / GKE / app in service project:
	•	Uses Shared VPC
	•	Is attached to the host VPC
	•	Connects to the PSC endpoint’s internal IP

👉 It does NOT connect to:
	•	The service attachment
	•	Nor directly to vendor load balancer IP

⸻

🔹 So What Does the Workload Need?

✅ It needs:
	•	Network connectivity to the subnet where PSC endpoint IP exists
	•	Firewall rule allowing traffic
	•	Proper DNS (if using DNS instead of raw IP)

❌ It does NOT need:
	•	Service attachment details
	•	Vendor load balancer IP

⸻

🔐 Important Notes
	•	PSC endpoint IP behaves like an internal load balancer IP
	•	Routing happens automatically within Shared VPC
	•	No extra peering required
	•	Service project just consumes the host VPC network

⸻

🎯 Final Answer

Your service project workload connects to:

✔️ PSC endpoint internal IP
❌ Not the service attachment
❌ Not vendor load balancer IP

⸻

If you want, tell me:
	•	Is it GKE or VM?
	•	Is PSC endpoint regional?
	•	Are you using DNS mapping?

I can validate the exact flow for Kafka + Pega scenario.
If your on-prem Kafka server is connecting to your vendor’s Pega Cloud using Private Service Connect (PSC) from your host project (Shared VPC), whether the application needs a service project depends on your network architecture.

Let’s break it down clearly:

⸻

🔹 Your Current Setup (Assumed)
	•	Kafka is on-prem
	•	Connected to Pega Cloud (vendor side)
	•	Using PSC endpoint created in your host project
	•	You are using Shared VPC architecture

⸻

✅ When You DO NOT Need a Service Project

You don’t need a separate service project if:
	•	The PSC endpoint is created in the host project
	•	The subnet used for PSC is in the host project
	•	The on-prem traffic comes through:
	•	Cloud VPN or
	•	Cloud Interconnect
	•	No GCP compute workload (GKE/VM/App) inside GCP needs to access PSC

In this case, the host project alone is enough.

⸻

✅ When You DO Need a Service Project

You need a service project if:
	•	Your application runs in:
	•	GKE
	•	Compute Engine
	•	Any workload inside GCP
	•	And that workload is attached to a Shared VPC
	•	And needs to access the PSC endpoint

In Shared VPC:
	•	Network resources (VPC, subnets, PSC endpoints) live in the host project
	•	Compute workloads live in the service project

So yes — if your application is deployed in GCP, it typically runs in a service project attached to the host project.

⸻

🔐 Important Security Check

If your concern is data leak or isolation:
	•	PSC endpoints are regional and VPC scoped
	•	Only resources in connected networks can access them
	•	IAM + firewall rules control access
	•	Service project workloads must be explicitly allowed via:
	•	Firewall rules
	•	Subnet routing

So adding a service project does NOT automatically create risk — access still must be allowed.

⸻

🎯 Final Answer (Short)
	•	❌ If Kafka is only on-prem → No service project needed
	•	✅ If an app inside GCP needs to access PSC → Yes, use a service project (with Shared VPC)

⸻

If you tell me:
	•	Where your Kafka runs (pure on-prem or hybrid?)
	•	Where your consuming application runs?
	•	Whether you are using Shared VPC officially?

I can give you a precise architecture recommendation.
