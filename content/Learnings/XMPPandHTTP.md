---
title: "De-mystifying XMPP: Insights from a Mentorship Session" 
date: 2026-02-14 
tags: [XMPP, Mentorship, OpenSource, Protocol] 
draft: false
---


I recently had the opportunity to sit down with my mentor, **Schimon Jehudah**, for an intensive technical session. We spent time deconstructing XMPP, and it was eye-opening to see how it compares to the traditional web architectures most of us are used to.

Here is a summary of the core concepts we discussed—perfect for anyone looking to understand what makes XMPP unique.

## XMPP vs. HTTP: A Different Philosophy

The biggest takeaway was how these protocols handle data. In the HTTP world, you typically rely on separate server-side software to manage your data.

However, modern XMPP servers like **eJabberd, Prosody, Tigase, and Jabberd2** are designed to integrate both communication and data management into a single, unified operating environment. This architecture allows the server to handle more than just passing messages; it manages the state and data of the application itself.

## The Foundations: RFCs vs. XEPs

We broke down the ecosystem into two main categories:

- **RFC (Request for Comments):** This is the "Communication Layer System." It’s the foundational manual that ensures the protocol works consistently across different platforms.
    
- **XEP (XMPP Extension Protocol):** These are modular extensions, similar to plugins. While RFCs handle the "how" of communication, XEPs handle the "what"—the specific data management practices.
    

## The Power of Modularity

XMPP is inherently **modular**. This means the system is built from independent parts that don't rely on each other to function. For developers, this is a huge advantage: you only implement the specific XEPs you need, keeping your build lean and highly customized.

## Data Formats: XML over JSON

While most web developers today gravitate toward JSON, XEPs explicitly define and prefer **XML** for managing and communicating data. This provides a robust, structured way to handle complex data across the network.

## Critiquing the Terminology

One of the most interesting parts of our discussion was how certain names in the XMPP world might actually hide their true potential from new developers:

- **PubSub vs. Storage:** **XEP-0060 (Publish-Subscribe)** is frequently used for data storage, but the name doesn't make that obvious. Many developers assume XMPP can't handle storage like HTTP simply because of this naming convention.
    
- **Microblogging vs. Publishing:** **XEP-0277** is titled "Microblogging," but because it uses the Atom Format, it can handle full-length articles or even e-books. The term "micro" is quite misleading for such a powerful publishing tool.
    

A shift toward terms like **"Storage"** and **"Publishing"** would likely help developers see the true utility of these extensions.

## A Note of Appreciation

A huge thank you to **Schimon Jehudah** for such an incredible learning session. Having a mentor who is willing to dive deep into the philosophy and "why" behind the technology makes a world of difference.

The clarity I gained from this meeting has already changed how I'm approaching my current projects. I’m incredibly grateful for the guidance and the expertise Schimon continues to share with me as I navigate the KDE community and open-source development!