# PSC
Great, now we’re on the producer side 👌
I’ll explain how you publish a service using “Published services” in PSC, step-by-step, in plain Cloud SQL / Apigee-style terms, but applied to your own service.

Think: “How do I become Google in this picture?”

⸻

Big picture first 🧠

When you publish a service, you are saying:

“I own a backend. I want other VPCs (or projects) to reach it privately via PSC.”

Publishing a service = creating a Service Attachment backed by a load balancer.

⸻

Step-by-step: Publishing a PSC service

I’ll map Console → What actually happens → Terraform names.

⸻

1️⃣ You must already have a backend service

PSC does NOT publish raw VMs.

Your backend must be one of:
	•	VM-based service
	•	GKE service
	•	On-prem service (via VPN / Interconnect)

Under the hood

You need:
	•	NEG (zonal or hybrid)
	•	Backend Service
	•	Health Check

➡ This is a real load balancer — owned by you.

⸻

2️⃣ Create the load balancer (producer-side)

This is the LB consumers will never see.

Typical choices:
	•	Internal TCP LB
	•	Internal HTTPS LB
	•	Internal SSL Proxy LB

Example flow

Client (PSC)
   ↓
Producer Load Balancer
   ↓
Backend Service
   ↓
NEG → VMs / Pods / On-prem


⸻

3️⃣ Create NAT subnet (mandatory)

Why this is required

PSC hides consumer IPs.

So Google needs a subnet to:
	•	Source-NAT incoming PSC traffic
	•	Present traffic to your backend

Key rules
	•	Regional
	•	Dedicated to PSC
	•	Only used for PSC traffic

This subnet appears in:

Published Service → NAT Subnets


⸻

4️⃣ Create the Service Attachment ⭐

This is what makes your service publishable.

Console action

Network services → Private Service Connect → Published services → Create

You must choose:
	•	Region
	•	Backend service
	•	NAT subnet
	•	Connection preference
	•	Allowed consumers

⸻

Under the hood

This creates:

google_compute_service_attachment

This is the actual published service.

⸻

5️⃣ (Optional) Attach a connection policy

Instead of managing allow-lists per service:
	•	Create Connection Policy
	•	Attach it to your published service

This controls:
	•	Which projects can connect
	•	Auto vs manual approvals

Enterprise-grade governance.

⸻

6️⃣ What consumers do (for context)

Consumers:
	•	See your service
	•	Create a connected endpoint
	•	Get a private IP
	•	Send traffic

They never touch your LB.

⸻

Full producer-side architecture

Producer Project
 ├── Internal Load Balancer
 │    ├── Target Proxy
 │    ├── Backend Service
 │    ├── Health Check
 │    └── NEG (VM / GKE / On-prem)
 │
 ├── NAT Subnet
 │
 ├── Service Attachment   ← Published Service
 │    └── Connection Policy


⸻

Why “Published Service” looks simple in UI

The UI hides:
	•	Load balancer complexity
	•	Backend health logic
	•	NEG wiring

Because consumers should not care.

⸻

Common mistakes (important ⚠️)

❌ Trying to publish a VM directly
❌ Forgetting NAT subnet
❌ Using global backend service
❌ Expecting consumer to see health checks

⸻

One-line summary 🧠

Publishing a service in PSC means exposing your load-balanced backend through a service attachment, so other VPCs can connect privately without peering.

⸻

If you want next:
	•	Terraform example (minimal, real)
	•	On-prem publishing (Hybrid NEG + Network Attachment)
	•	Why PSC forces regional LBs
	•	Debug checklist when consumers can’t connect

Just tell me 👍
