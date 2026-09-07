# DEDUCT: A Secure Auditing and De-Duplication of Data in Cloud

DEDUCT is a secure deduplication framework for textual data in cloud storage. It combines strong encryption with hash-based duplicate detection so that cloud providers can eliminate redundant files **without ever seeing the plaintext content**, and lays the groundwork for detecting duplicates that are semantically similar (paraphrased) rather than byte-for-byte identical.

## Problem

Cloud storage grows explosively, and a large share of it is redundant text — repeated emails, reports, and logs. Standard deduplication saves space but usually requires access to plaintext, which is a privacy and security risk. Existing secure-deduplication schemes also tend to:
- rely on a trusted third party / central key server (single point of failure)
- only catch *exact* duplicates, missing reworded or paraphrased content
- carry heavy cryptographic overhead that doesn't scale well to many users or low-power devices

## Approach

DEDUCT addresses this with:
- **Encryption before storage** — files are encrypted (AES, with a DES-based double-encryption variant explored for comparison) using client-held keys, so the cloud never sees raw content.
- **Hash-based deduplication** — an encrypted file's hash/checksum is checked against existing hashes; duplicates are flagged and the upload is skipped, saving storage and bandwidth.
- **Key Distribution Center (KDC)** — issues keys to authenticated clients at setup time, then steps out of the pipeline (not needed for every operation).
- **Pointer-based storage** on the Cloud Service Provider (CSP) side to manage duplicates efficiently.
- Designed to extend toward **semantic-aware deduplication** (NLP-based similarity, e.g. embeddings + cosine similarity) so paraphrased/reworded documents can also be recognized as redundant.

## Architecture

**Actors:** Authorized Clients, Key Distribution Center (KDC), Cloud Service Provider (CSP)

**Flow:**
1. Client authenticates and obtains an encryption key from the KDC.
2. Client encrypts the file locally (AES, with IV generation and padding).
3. Client computes a hash/checksum of the encrypted file.
4. The Deduplication Checker compares the hash against existing records in cloud storage.
5. **Duplicate found →** upload skipped, user notified.
6. **Unique file →** encrypted file + IV uploaded and stored.
7. On download, the encrypted file + IV are retrieved and decrypted locally with the original key.

## ⚙️ Tech Stack

| Component | Technology |
|---|---|
| Language | Java (JDK 8+) |
| IDE | NetBeans |
| App Server | Apache Tomcat |
| Database | MySQL (deduplication metadata: hashes, keys, user info) |
| Crypto | AES (CBC/GCM), SHA-256 hashing, DES/DDDES (comparative study) |
| Libraries | MySQL Connector/J, Apache Commons Codec, JavaMail, Apache FTPClient |

## Results

- Compared **DES-64, AES-128, and DDDES-64** on compression ratio and bandwidth usage — DDDES-64 gave the best compression ratio and lowest bandwidth usage among the three.
- Compared the proposed system against a baseline on performance and latency — the proposed design showed markedly higher performance and lower latency.
- Full test suite covering unique-file upload, duplicate detection, edge cases (empty files, invalid keys, large files), and correct decryption on retrieval.

## Future Scope

- Semantic deduplication via NLP (detecting reworded/paraphrased duplicates, not just exact matches)
- Blockchain-based multi-user access control and immutable audit logging
- Real-time auditing and alerts for suspicious access
- Extending to multimedia (images/video/audio) via perceptual hashing
- Cross-cloud interoperability (AWS, Azure, GCP)
- Lightweight variants for edge/IoT devices
