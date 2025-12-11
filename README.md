# BYOC-Cloud-with-Azure-Audiocodes-SBC
Deployed SBC audiocodes on Azure cloud. Buy DID number from Twilio Carrier. Create trunk SIP on Genesys cloud, SBC and Twilio.

This project demonstrates how to set up end-to-end telephony using Genesys Cloud and SBC Audiocodes on Azure and Twilio is a Carrier DID. Call routing, WebRTC phones, Architect flows, and inbound call handling and outbound call handling in Genesys Cloud. 

**Genesys BYOC (AudioCodes on Azure) — Twilio DID integration**

BYOC setup: AudioCodes SBC on Azure ↔ Genesys Edge ↔ Genesys Cloud and AudioCodes SBC ↔ Twilio SIP Trunk (DID). I’ve included both inbound (caller → Twilio DID → Genesys IVR) and outbound (agent → Genesys → Twilio DID) call flows, AudioCodes config examples (web/CLI style), routing/dial-plan examples

**1) High-level architecture / components**

Twilio — owns the DID (public number). SIP Trunk (origination/termination) points to your AudioCodes SBC.

Public Internet / Azure NSG / Firewall — transport between Twilio and your SBC.

AudioCodes SBC (on Azure VM) — terminates Twilio SIP trunk and terminates SIP trunk toward Genesys Edge. Performs NAT, TLS/SRTP, codecs, routing rules, authentication, SIP header manipulations.

Genesys Edge — connects to your SBC (SIP trunk) and to Genesys Cloud control plane. Edge will route calls into your Genesys Cloud environment (IVR / queues / agents).

Genesys Cloud — IVR flows, queues, agents — inbound/outbound call control.

**2) Recommended transport / security**

Signaling: use TCP/UDP on 5060 between Twilio ↔ SBC ↔ Genesys Edge where possible. Twilio supports TCP/UDP ; prefer TCP/UDP  to avoid NAT issues and for security.

Media: use RTP  end-to-end if Genesys Cloud & AudioCodes support it. 

Certificates: use publicly trusted certs on the SBC TLS interface (or certs that Genesys Edge / Twilio will accept). CN/SAN should match the FQDN that peers use.

SIP ALG: disable SIP ALG on any NAT devices in front of SBC (Azure NSG, virtual appliances).

Allowed IPs: whitelist Twilio IP ranges and Genesys Edge IPs in Azure NSG/firewall.

Health checks: configure SIP OPTIONS / keepalive between SBC and Twilio and SBC and Genesys Edge.

Codec negotiation: allow G.711ulaw, G.711alaw, opus (if supported), G.729 (license required). Prefer G.711 for PSTN DID

**3) Twilio SIP trunk — setup notes (high-level)**

In Twilio Console:

Create SIP Trunk (or use Elastic SIP Trunking).

Origination: point Twilio to your SBC public IP / FQDN (SBC_PUBLIC_IP:5060) — this is where inbound calls go.

Termination: add credential list (username/password) or IP Access Control List (Twilio will use your SBC IP to accept termination). For secure operation, prefer TCP and IP ACLs.

DID / Phone Number: assign the purchased phone number to the SIP trunk.

**4) Genesys Cloud / Edge — trunk & routing (high-level steps)**

In Genesys Admin UI:

Create a SIP Trunk (BYOC / External Trunk) that represents your SBC toward Genesys Edge.

Configure Edge to accept/establish TCP connection to the SBC (Edge configuration for SIP trunking).

Create Inbound route:

Map the incoming DID (TWILIO_DID) to your IVR flow (Interaction Router / Architect).

Create Outbound Trunk:

Configure dialing rules so agent outbound calls to E.164 are sent to the SBC trunk and the SBC will forward to Twilio.

Test with a dummy user/queue and log calls.

** Quick checklist before going live**

 TLS certs installed & verified

 Twilio trunk origination points to SBC (and Twilio verifies cert / IP)

 Genesys Edge trunk created & reachable

 Codecs negotiated and verified

 SIP OPTIONS health checks configured

 Monitoring/alerts & logging in place (SBC logs, Twilio Debug, Genesys logs)

 Failover path or dial-plan for trunk failover



