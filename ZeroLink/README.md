# ZeroLink – Offline UPI Payment Mesh

ZeroLink is a **Spring Boot-based offline payment prototype** that demonstrates how payments can travel between devices without requiring an active internet connection on every device.

It uses a simulated Bluetooth-style mesh network to transfer encrypted payment packets between devices. When a device with internet access receives the packet, it forwards it to the backend for verification and settlement.

The project focuses on secure communication, duplicate transaction prevention and reliable payment processing.

> **Note:** ZeroLink is a software-based prototype. It simulates mesh communication and does not perform real UPI transactions or use actual Bluetooth hardware.

---

## Dashboard Preview

The interactive dashboard provides a view of the complete payment process, from payment creation to final settlement.

![ZeroLink Dashboard](docs/images/zerolink-dashboard.png)

**Dashboard features:**
- Create and inject payment packets into the mesh.
- Simulate packet propagation between devices.
- Upload packets through bridge nodes.
- Monitor device connectivity and packet status.
- View account balances and transaction history.
- Track system events through the activity log.

---

## Key Features

- **Offline Payment Simulation:** Transfers payment packets across virtual devices without requiring internet connectivity at every node.
- **Hybrid Encryption:** Protects payment information using RSA-OAEP and AES-256-GCM.
- **Mesh Network Simulation:** Uses gossip-based propagation to distribute packets between devices.
- **Duplicate Prevention:** Uses SHA-256 hashing and atomic operations to prevent duplicate transaction processing.
- **Replay Protection:** Validates transaction timestamps and unique nonces.
- **Secure Settlement:** Uses database transactions to maintain consistent debit and credit operations.
- **Concurrency Handling:** Uses ConcurrentHashMap and optimistic locking to handle simultaneous requests.

---

## Technology Stack

| Category | Technologies |
|---|---|
| Language | Java 17 |
| Backend | Spring Boot 3.3 |
| APIs | REST APIs |
| Build Tool | Maven |
| Database | H2 (In-Memory) |
| ORM | Spring Data JPA, Hibernate |
| Security | RSA-OAEP, AES-256-GCM, SHA-256 |
| Concurrency | ConcurrentHashMap, Optimistic Locking |
| Frontend | HTML, CSS, JavaScript |
| Testing | JUnit |

---

## How It Works

ZeroLink follows a four-step payment process.

**1. Create Payment**

The sender initiates a payment by providing the sender, receiver, amount and PIN. The system creates a payment instruction with a unique nonce and timestamp.

**2. Encrypt and Propagate**

The payment instruction is encrypted using AES-256-GCM, while the AES key is protected using RSA-OAEP.

The encrypted packet enters the simulated mesh network, where virtual devices exchange packets through gossip-based propagation. TTL (Time to Live) limits how far a packet can travel.

**3. Bridge Node Upload**

A bridge node represents a device that regains internet connectivity. It forwards the stored payment packets to the backend through a REST API.

**4. Verify and Settle**

The backend:
- Generates a SHA-256 hash of the ciphertext.
- Checks for duplicate packets.
- Decrypts and validates the payment.
- Checks transaction freshness.
- Debits the sender and credits the receiver.
- Records the transaction in the ledger.

---

## System Architecture

```text
       SENDER DEVICE (OFFLINE)
                  |
                  v
         Payment Instruction
                  |
                  v
          Hybrid Encryption
             RSA + AES
                  |
                  v
             Mesh Packet
                  |
                  v
          Simulated Mesh
                  |
          Gossip Propagation
                  |
                  v
             BRIDGE NODE
          (Internet Available)
                  |
                  v
             Backend API
                  |
                  v
           SHA-256 Hashing
                  |
                  v
           Idempotency Check
                  |
                  v
          Decryption & Validation
                  |
                  v
          Transaction Settlement
                  |
                  v
             Account Ledger
```

---

## Key Technical Implementations

### 1. Hybrid Encryption

**Problem:** Intermediate devices should not be able to read or modify payment information.

**Solution:**
- AES-256-GCM encrypts the payment data.
- RSA-OAEP encrypts the AES key.
- AES-GCM authentication detects ciphertext tampering.

Only the server holding the corresponding private key can decrypt the payment payload.

### 2. Idempotency and Duplicate Prevention

**Problem:** Multiple bridge nodes may upload the same payment packet simultaneously.

**Solution:**

ZeroLink calculates a SHA-256 hash of the ciphertext and uses:

`ConcurrentHashMap.putIfAbsent()`

This atomic operation allows only the first request to proceed. Duplicate requests are rejected before settlement.

A unique database constraint on the transaction packet hash provides an additional safeguard.

### 3. Replay Attack Protection

**Problem:** An attacker could attempt to resubmit an old payment packet.

**Solution:**
- Validates that the transaction timestamp is within 24 hours.
- Uses a unique nonce for each payment.
- Uses ciphertext-based idempotency checks to reject repeated packets.

### 4. Concurrent Settlement

**Problem:** Simultaneous payment requests can cause conflicting account updates.

**Solution:**
- `@Transactional` ensures debit and credit operations are handled together.
- JPA `@Version` provides optimistic locking for account updates.

---

## Getting Started

### Prerequisites

- Java JDK 17 or later
- Git
- Internet connection for the initial dependency download

Maven does not need to be installed separately because the project includes Maven Wrapper.

### 1. Clone the Repository

```bash
git clone <YOUR_REPOSITORY_URL>
cd ZeroLink
```

### 2. Run the Application

**Windows (PowerShell):**

```powershell
.\mvnw.cmd spring-boot:run
```

**Mac/Linux:**

```bash
./mvnw spring-boot:run
```

### 3. Open the Dashboard

Once the application starts, visit:

http://localhost:8080

### 4. Run Tests

**Windows:**

```powershell
.\mvnw.cmd test
```

**Mac/Linux:**

```bash
./mvnw test
```

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | Opens the dashboard |
| GET | `/api/server-key` | Retrieves the server's RSA public key |
| GET | `/api/accounts` | Retrieves account balances |
| GET | `/api/transactions` | Retrieves recent transactions |
| GET | `/api/mesh/state` | Retrieves mesh device states |
| POST | `/api/demo/send` | Creates and injects a payment packet |
| POST | `/api/mesh/gossip` | Runs a mesh propagation round |
| POST | `/api/mesh/flush` | Uploads packets through bridge nodes |
| POST | `/api/mesh/reset` | Resets the mesh and cache |
| POST | `/api/bridge/ingest` | Receives and processes payment packets |

---

## Testing

The project includes tests for important transaction and security scenarios:

- **Encryption and Decryption:** Verifies successful encryption and decryption.
- **Tampering Detection:** Checks whether modified ciphertext is rejected.
- **Concurrent Idempotency:** Simulates three simultaneous submissions of the same packet and verifies that only one transaction is settled.

Run the concurrency test:

```powershell
.\mvnw.cmd test -Dtest=IdempotencyConcurrencyTest#singlePacketDeliveredByThreeBridgesSettlesExactlyOnce
```

---

## Current Limitations

ZeroLink is a proof-of-concept implementation. The following components are simulated or simplified:

- Mesh communication uses virtual devices instead of actual Bluetooth hardware.
- H2 in-memory database is used instead of a persistent production database.
- Idempotency uses an in-memory ConcurrentHashMap instead of distributed Redis.
- RSA keys are generated when the server starts.
- The system is not integrated with NPCI, banks or actual UPI infrastructure.
- Offline balance verification and double-spending prevention are not fully implemented.

---

## Future Improvements

- Implement actual Bluetooth Low Energy (BLE) communication between Android devices.
- Develop an Android application for sending and receiving payments.
- Integrate Redis for distributed idempotency management.
- Introduce hardware-backed key storage and device authentication.
- Explore secure offline balance management and double-spending prevention.

---

## Project Objective

ZeroLink explores how payment information can be securely transferred across devices in areas with limited or no internet connectivity.

It demonstrates the use of **cryptography, distributed communication, idempotency and transaction management** to build a prototype for offline payment propagation and deferred settlement.