---
icon: lucide/rocket
title: Introduction
---

# Introduction

Welcome to the **Aurix Integration Hub**. This guide helps partners and developers connect with the Aurix platform.

## Collaboration

This documentation is built for our partners. We know every system is different, and we are ready to adapt.

> **Our Commitment**: If specific endpoints, data structures, or webhooks are identified that would streamline the integration, please contact our development team. We are eager to collaborate and implement solutions that ensure the integration goes smoothly.


## About Aurix

Aurix enables businesses to connect their social media accounts to intelligent AI tools. Once connected, we can securely push and share collected data, such as qualified leads and widget interaction details,directly with your system in real time. Our API and webhooks enable seamless synchronization, allowing your platform to immediately receive and act on new information gathered by our AI agents.


*   **Lead Gen Agent**: Automatically collects and qualifies user details via chat.
*   **FAQ Agent**: Instantly answers customer questions using a verified knowledge base.

## WhatsApp Widget

The **WhatsApp Widget** embeds a chat widget on websites. It links web traffic data (like UTM parameters) to WhatsApp conversations to track attribution from ad clicks to messages.

**How it Works:**

1.  **Captures Traffic Data**: A lightweight script on the website captures URL parameters (UTM, GCLID), source URLs, and device identifiers.
2.  **Generates Correlation**: When a user clicks to chat, the system generates a unique **Correlation ID** (e.g., `O-xY123z`).
3.  **Links Conversation**: This ID is embedded in the pre-filled WhatsApp message. When the user sends it, the web session data is permanently linked to the new lead.

!!! warning "Development Status"
    The WhatsApp widget is currently in **active development**.
    
    *   **Now**: Supports URL parsing, click tracking, and basic correlation.
    *   **Coming Soon**: Advanced device tracking, network and page context.

## Current vs Planned Tracking Capabilities

| Feature | Status | Details |
| :--- | :---: | :--- |
| **Raw IP Address** | :lucide-check: | **Available.** |
| **Raw User-Agent** | :lucide-check: | **Available.** |
| **Landing Page URL** | :lucide-check: | **Available.** |
| **Parsed Ad Params** | :lucide-check: | **Available.** <br>• Full **"Capture Everything"** approach with raw parsing. <br>• **No Truncation:** We do not filter or truncate keys. We store every parameter as presented in the URL.<br>• **Duplicate Handling:** Automatically converts duplicate keys (e.g., multiple `utm_source`) into arrays (`["google", "search"]`). |
| **Network** | :lucide-x: | **Not available.** (Carrier/ASN/Geo-location). |
| **Device Fingerprinting** | :lucide-x: | **Not available.**  |
| **Page Context** | :lucide-x: | **Not available.** (Referrer URLs and Page Titles are not currently captured). |
