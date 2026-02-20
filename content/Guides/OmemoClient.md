---
title: "Guide: Building and Using the OMEMO Sync Client"
date: 2026-02-20
tags: [KDE, Falkon, Python, Qt, OpenSource]
draft: false
---


To understand the code, you have to understand these four pillars:

- **`QXmppOmemoManager`**: This is the "Encryption Engine." It handles the **Double Ratchet** algorithm, which constantly changes the encryption keys for every message so that even if one key is stolen, the rest of the conversation stays safe.
    
- **`PubSub` (XEP-0060)**: OMEMO doesn't send keys directly to people. It "publishes" them to a node on the server. Think of it like a public locker where your bot can go to pick up the "lock" (Public Key) needed to send you a message.
    
- **`Toakafa`**: This is your Trust Policy. In your code, it means "Trust Once and Keep Always For All." It tells the plugin to automatically accept encryption keys from any device logged into your account.
    
- **`sendSensitive`**: This is a specialized function. Instead of just sending a message, it tells the client to:
    
    1. Look up the recipient's OMEMO keys.
        
    2. Encrypt the text.
        
    3. Wrap it in a specific OMEMO XML tag.
        

---

## 2. The Full Implementation (`XmppSync.cpp`)

Here is your code with added comments explaining exactly what each section is doing for the OMEMO setup.

C++

```cpp
#include "XmppSync.h"
#include <QXmppOmemoManager.h>
#include <QXmppOmemoMemoryStorage.h>

void XmppSync::init(InitState state, const QString &settingsPath)
{
    // 1. Initialize the Client
    m_client = new QXmppClient(this);

    // 2. Setup Storage (The "Vault")
    // We use MemoryStorage, meaning keys are lost when Falkon closes.
    m_trustStorage = std::make_unique<QXmppAtmTrustMemoryStorage>();
    m_omemoStorage = std::make_unique<QXmppOmemoMemoryStorage>();

    // 3. Attach the OMEMO Manager
    m_omemoManager = m_client->addNewExtension<QXmppOmemoManager>(m_omemoStorage.get());
    
    // Tell the client to use OMEMO for sensitive data
    m_client->setEncryptionExtension(m_omemoManager);

    // 4. Handle Connection
    connect(m_client, &QXmppClient::connected, this, [this]() {
        // Start the OMEMO "Handshake" - generating keys and announcing to server
        auto setupFuture = m_omemoManager->setUp(QStringLiteral("Falkon_Sync_Device"));

        setupFuture.then(this, [this](bool success) {
            if (success) {
                // Set the 'Trust' policy so we can talk to our bot automatically
                m_omemoManager->setSecurityPolicy(QXmpp::Toakafa);
                qDebug() << "OMEMO Ready!";
            }
        });
    });

    m_client->connectToServer(QStringLiteral("user"), QStringLiteral("pass"));
}

// 5. Sending an Encrypted Sync Message
void XmppSync::sendSecretMessage(const QString &jid, const QString &text) 
{
    QXmppMessage msg;
    msg.setTo(jid);
    msg.setBody(text);

    // CRITICAL: sendSensitive triggers the OMEMO encryption layer
    m_client->sendSensitive(std::move(msg), QXmppSendStanzaParams());
}
```

---

## 3. How to use it with a Bot

If you want to sync your bookmarks, you need a **Bot** on the other end to receive them.

### Step 1: The Bot Identity

The Bot must also support OMEMO (using a library like `python-omemo` or another `QXmpp` instance). It needs to be logged in as the JID you defined in your code (e.g., `user@xmpp.jp`).

### Step 2: The Handshake

When your Falkon plugin starts, it sends a "Test Message" after 5 seconds.

1. **Falkon** asks the server: "Does `user@xmpp.jp` have OMEMO keys?"
    
2. **Server** sends back the Bot's keys.
    
3. **Falkon** encrypts the message and sends it.
    

### Step 3: Verifying the Sync

In your `onMessageReceived`, you verify the bot's response like this:

C++

```cpp
void XmppSync::onMessageReceived(const QXmppMessage &message)
{
    // Check if the message has E2EE metadata (it was encrypted)
    if (message.e2eeMetadata().has_value()) {
        qDebug() << "Secure sync data received from bot:" << message.body();
        // Now you can safely parse message.body() as JSON bookmarks
    } else {
        qWarning() << "Unsecured message received! Ignoring for safety.";
    }
}
```

---

## 4. Summary for New Developers

1. **To Build**: You need `QXmppQt6` and `QXmppOmemoQt6` installed.
    
2. **To Test**: You need an OMEMO-capable XMPP account.
    
3. **The Goal**: Use `sendSensitive` to transmit bookmark data so that nobody (not even the XMPP server admin) can see your browser history.

## 5. Device Trust: Securing the Session

For encryption to work, your devices must be in a "Secured" state. In OMEMO, you don't just trust an _account_; you trust specific **Device IDs**.

### What is a "Secured" Session?

A session is active when your Falkon client and the Bot have successfully exchanged "Pre-Keys." If the session isn't secured, the `message.body()` will appear as garbled ciphertext or remain empty because the library can't find a valid key to unlock it.

### How your code handles Trust

You are using **ATM (Automatic Trust Management)** with a specific policy:

C++

```cpp
// TOAKAFA: Trust Once And Keep Always For All
m_omemoManager->setSecurityPolicy(QXmpp::Toakafa);
```

- **Why this matters:** Without this line, the developer would have to manually "Verify" every device's fingerprint (like scanning a QR code).
    
- **The "Secured" Logic:** By setting this to `Toakafa`, your code tells the OMEMO manager: _"If I see a new device from my sync partner, trust it automatically and start a secure session."_
    

### Checking if a Device is "Okay"

When a message arrives, you should verify if it came from a trusted source before processing sensitive sync data:

C++

```cpp
void XmppSync::onMessageReceived(const QXmppMessage &message)
{
    // 1. Verify it is an OMEMO message
    if (!message.e2eeMetadata().has_value()) {
        qWarning() << "Insecure message received. Dropping!";
        return;
    }

    // 2. Check the Device ID and Trust Level
    quint32 senderDeviceId = message.e2eeMetadata()->deviceId();
    auto trustLevel = m_omemoManager->trustLevel(message.from(), senderDeviceId);

    if (trustLevel == QXmpp::TrustLevel::Trusted) {
        qDebug() << "Session is Secured. Device ID:" << senderDeviceId;
        qDebug() << "Decrypted Content:" << message.body();
    } else {
        qWarning() << "Message received from an untrusted device. Decryption may fail!";
    }
}
```

---