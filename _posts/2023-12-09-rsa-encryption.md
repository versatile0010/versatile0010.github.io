---
title: " 🔐 [en] Enhancing API Security with RSA Encryption in Spring/Java and Redis "
layout: single
classes: wide
categories:
  - Security
---

> To protect sensitive information in API communication, RSA encryption is can be used.
And redis helps manage keys quickly and efficiently.

<div style="text-align: center;">
    <img src="https://github.com/user-attachments/assets/9e02c973-31b7-4f73-bbbe-855ceec7667f" alt="image" width="600">
</div>

<br>

### When developing a service, it's important to think about performance but it's also very important to find ways to better protect the sensitive data used in API communications.

OK, let's assume we have implemented a login feature.

The `Client` will receive the user's `ID` and `Password`, then include them in the HTTP Payload to call the Login API.

Typically, sensitive information like `password` is displayed with masking as shown below (front-end side)

However, without additional security measures, all raw data in the payload can be exposed in developer tools!

<img width="364" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/0f058b69-22e5-45d3-aacb-aa0210cc1431">

When the back-end server sends data to the client, if security is not handled properly,

sensitive information might be shown in the response tab.

If customers knew it, they may lose trust and stop using our service.

If we keep ignoring this issue, it could cause serious problems, even if the chance is low...

---

### How Can We Solve This?

Actually, the solution is easy to think of.

We can encrypt sensitive information so that others cannot see it.

Let's Roughly Outline the Requirements.

- When the client sends sensitive information to the server, encrypt it. Then, the server should decrypt it and perform the necessary logic.

If we can meet these requirements, the previously mentioned problem will be solved.

Now that the problem is defined, let's think about which tools to use to solve it.

---

### Which Encryption Algorithm Should We Use?

Encryption algorithms can be broadly classified as follows:

```java
1. One-Way Encryption(단방향 암호화)
2. Symmetric Key Encryption(대칭키 암호화)
3. Asymmetric Key Encryption(비대칭키 암호화)
```

One-way encryption `hashes` the plain text and cannot be decrypted, which is why it is mainly used for encrypting passwords.

To meet our requirements, we need to be able to decrypt the data.

Therefore, one-way encryption is excluded from our options.

The following are symmetric key encryption and asymmetric key encryption.

To explain roughly:

- Symmetric key encryption can only be decrypted with the key used for encryption.
  
- e.g) AES Algorithm

<img width="442" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/6d2c49a4-292b-4bd9-a437-7462acb63c8a">

- Asymmetric key encryption uses different keys for encryption and decryption.

- e.g) RSA Algorithm

<img width="441" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/ff7cbb58-2e2c-4646-95ba-2d6d2ab77c25">

Both Symmetric and Asymmetric Key Encryption Allow Decryption.

Which encryption algorithm should we use?

Implementing a symmetric key encryption algorithm (e.g AES) is much easier than implementing as asymmetric key encryption algorithm like RSA.

- (fun fact. we already had an AES Utils class implemented in our project, so we could have just used it)

However, we chose the RSA encryption algorithm for the following reasons:

1. To use symmetric key encryption(e.g. AES), you need to share the key with client-sde
2. With asymmetric key encryption(e.g. RSA), you don't need to share the same key. Anyone can encrypt data, but only authorized parties can decrypt it. 

Let's Check if we can meet our initial requirements with RAS.

- When the client-side sent sensitive information to the server, encrypt it. Then, the server decrypts it and performs the necessary logic.
  - If the client-side encrypts the data using the public key and send it to the server, and the server has the corresponding private key, this is possible. 

In other words, if the client encrypts the data with the public key and sends it to the server, and the server knows the corresponding private key, it works.

---

How can we implement This?

It's best to manage the RSA key pair securely on the server-side.

So, how about doing the following?

1. If the client needs to encrypt data, it requests an RSA public key from the server.
2. Backend server generates an RSA key pair and respond the public key to the client.
3. The client encrypts the data using the public key and sends the ciphertext along with the public key back to the server.
    - Even if a third-party intercepts this process, they cannot access the private key that matches the public key, so it's safe.
4. The server finds the private key that matches the public key, decrypts the data, performs the necessary logic, and then removes the key pair.

To achieve this, the server-side needs to be able to retrieve the private key corresponding to the public key encrypted by the client.

We can store these keys in a relational database(RDB), But using a Redis is more suitable because it offers faster I/O, allows setting a TTL and the operation to find a private key for a public key is expected to occur frequently.

Therefore, we decided that Redis, a key-value storage system, is more appropriate.

But, Let's consider one more thing (for optimization)

With the above method, the server needs to generate an RSA key pair for each request.

Let’s examine the cost of RSA key pair generation.

```java
Finding two large prime numbers	
  : O((log n)^2)
Computing the modulus	
  : O(log^2 n)
Computing modular inverses	
  : O(log^3 n)
Total
  : O((log n)^3)
```

Ok, it's not light-weight.

Do we really need to generate a new RSA Keypair for every request?

How about pre-generating several RSA Keypairs on the server, like a `thread pool`, and when a client request comes in,

randomly pick one and just send it?

<img width="1015" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/ec6ccf90-dbef-4fbf-bbda-7b7ece012da2">

1. If the client needs to encrypt data, it requests an RSA public key from the server.
2. Server-side selects one of the `pre-generated RSA key pairs and just sends the public key to the client.`
3. The client encrypts the data using the public key and sends the ciphertext along with the public key to the server.
   - Even if a third party intercepts this process, they cannot access the private key that matches the public key, so it's safe.
4. Server-side finds the private key that matches the public key, decrypts the data, and then performs the necessary operations.

But, there's one more thing to consider!

These process relies heavily on Redis.

What if the Redis server goes down?

Since we haven't considered this, all APIs using Redis would fail...

Although it's rare for the Redis server to go down, it's not impossible.

How can we prevent this?

We need to have a separate fallback so that our system works well even if the Redis server is not working.

---
## References
- https://www.baeldung.com/java-rsa
- https://www.geeksforgeeks.org/how-to-generate-large-prime-numbers-for-rsa-algorithm/
- https://dev.gmarket.com/47
