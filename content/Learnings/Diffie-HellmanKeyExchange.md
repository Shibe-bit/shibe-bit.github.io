---
title: "Building a Secret with Math: The Diffie-Hellman Guide"
date: 2026-02-19
tags: [Cryptography, Math, Engineering, Security]
draft: false
---

## The Variables

To start a handshake, we need two public numbers that everyone knows:

- **Base ($g$):** $2$
    
- **Modulus ($p$):** $19$
    

## Step 1: The Private Secrets

Two parties, **Alice** and **Shiva**, choose secret numbers (Private Keys). These are never shared with anyone.

- **Alice** chooses: $a = 8$
    
- **Shiva** chooses: $b = 5$
    

## Step 2: The Public Exchange

Both parties perform a calculation: $Base^{Secret} \pmod{Modulus}$ and send the result to each other.

- **Alice calculates:** $2^8 \pmod{19}$
    
    - $256 \div 19 = 13$ with a remainder of **$9$**.
        
    - Alice sends **$9$** to Shiva.
        
- **Shiva calculates:** $2^5 \pmod{19}$
    
    - $32 \div 19 = 1$ with a remainder of **$13$**.
        
    - Shiva sends **$13$** to Alice.
        

## Step 3: Creating the Shared Secret

Now, they take the number they received and raise it to their own private secret.

- **Alice receives $13$ and calculates:** $13^8 \pmod{19}$
    
    - Result = **$7$**
        
- **Shiva receives $9$ and calculates:** $9^5 \pmod{19}$
    
    - Result = **$7$**
        

**The Shared Secret is $7$.** Both parties arrived at the exact same number without ever telling each other their secrets ($8$ and $5$).

---

## Why is this Secure?

An eavesdropper (or the server) sees the following numbers:

- Base: $2$
    
- Mod: $19$
    
- Alice's Result: $9$
    
- Shiva's Result: $13$
    

To steal the secret, the eavesdropper has to solve for $x$ in the equation: $2^x \pmod{19} = 9$.

### The "Trapdoor" Logic

1. **With small numbers ($19$):** You can easily "loop" through and find that $2^8 = 256$, which satisfies the equation.
    
2. **With real-world numbers:** In protocols like OMEMO, the modulus is not $19$; it is a number with **77 digits**.
    

Even if a computer can do trillions of calculations per second, the "random" jumping of the numbers caused by the **Modulo** operation means there is no shortcut. The computer has to guess every single possibility, which would take longer than the age of the universe.

---

## Conclusion

Diffie-Hellman is the reason you can trust your data on an open network. By using massive prime numbers, we turn a simple math problem into an unbreakable digital vault.