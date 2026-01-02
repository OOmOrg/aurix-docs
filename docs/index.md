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

Aurix enables businesses to link their social media accounts to intelligent AI tools. Once connected, our API and webhooks allow external systems to use our specialized agents to automate tasks:

*   **Lead Gen Agent**: Automatically collects and qualifies user details via chat. It sends structured lead data to your system based on your settings.
*   **FAQ Agent**: Instantly answers customer questions using a verified knowledge base.

## WhatsApp Widget

The **WhatsApp Widget** embeds a chat interface on websites. It links web traffic data (like UTM parameters) to WhatsApp conversations to track attribution from ad clicks to messages.

**How it Works:**

1.  **Captures Traffic Data**: A lightweight script on the website captures URL parameters (UTM, GCLID), source URLs, and device identifiers.
2.  **Generates Correlation**: When a user clicks to chat, the system generates a unique **Correlation ID** (e.g., `O-xY123z`).
3.  **Links Conversation**: This ID is embedded in the pre-filled WhatsApp message. When the user sends it, the web session data is permanently linked to the new lead.

!!! warning "Development Status"
    The WhatsApp widget is currently in **active development**.
    
    *   **Now**: Supports URL parsing, click tracking, and basic correlation.
    *   **Coming Soon**: Advanced device tracking, session linking, and cross-platform identity matching.

## Data & Attribution Roadmap

We are upgrading our tracking to provide precise attribution data. While we currently provide raw request data, we are adding deeper intelligence to our data feeds.

Here is what is coming:

1.  **Network & Location**
    *   **Now**: Raw IP address.
    *   **Future**: Detailed location (City, Lat/Long), Carrier (e.g., Singtel), and connection type. This improves tracking accuracy.

2.  **Device & Browser Details**
    *   **Now**: Raw User-Agent string.
    *   **Future**: Specific details on Operating System (e.g., Android 10) and Browser (e.g., Chrome 143). This helps match browser clicks to server conversions.

3.  **Page Context**
    *   **Now**: Landing Page URL.
    *   **Future**: Full page titles and referrer data (e.g., "YouTube" vs "Google").


