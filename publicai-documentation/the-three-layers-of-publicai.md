---
hidden: true
---

# 📖 The Three Layers of PublicAI

<figure><img src=".gitbook/assets/77 System Architecture2.png" alt=""><figcaption><p>The Three Layers of PublicAI</p></figcaption></figure>

**DataHub (1st Layer)**

* **Purpose**: The DataHub is the main platform for data campaigns and quality validation, where **uploaders** and **voters** collaborate to ensure high-quality data for AI training.
* **Core Functions**:
  * Hosts **data collection campaigns**, allowing uploaders to contribute datasets aligned with campaign instructions.
  * Enables **voters** to assess the quality of uploaded data, ensuring alignment with specified requirements.
  * Utilizes a **voting-based consensus mechanism** to maintain data quality and integrity.
* **Key Features**:
  * **Uploaders** participate in campaigns to earn **USDT** for high-quality contributions.
  * **Voters** assess whether uploaded data meets campaign instructions by voting on its quality.
  * **Rewards** contributors and voters based on their participation and alignment with the consensus (e.g., "hit" or "miss" the consensus).

***

**Data Hunter (2nd Layer)**

* **Purpose**: Data Hunter is now focused on enabling **node operators** to support AI agents, improving data processing and community engagement.
* **Core Functions**:
  * Allows **node operators** to run AI agents via the **Data Hunter extension**, contributing computational resources for data quality checks in the DataHub.
  * Facilitates active participation in the PublicAI ecosystem through tasks like replying to popular **X (formerly Twitter)** posts using AI(Reply to any popular post with views>1k).
*   **Key Features**:

    * **Node Operators**:
      * Provide computational resources to enhance AI agent efficiency, accelerating data validation in the DataHub.
      * Use the **“AI Reply”** feature to interact with popular X posts, generating valuable AI feedback data.



***

**Blockchain and Smart Contracts (3rd Layer)**

* **Purpose**: The blockchain layer ensures security, transparency, and fairness in managing data contributions and rewards.
* **Core Functions**:
  * Uses **smart contracts** to manage tasks like voter consensus, uploader rewards, and node operator incentives.
  * Tracks and secures all activities, ensuring data provenance and immutability.
  * Enables decentralized governance, allowing community members to influence platform developments.
* **Key Features**:
  * **Consensus Mechanism**: Implements a **Byzantine Fault Tolerance (BFT)** algorithm to verify data quality through decentralized voting.
  * **Incentives and Penalties**: Rewards loyal contributors and penalizes malicious behavior using automated blockchain mechanisms.
  * **Transparency**: Ensures all data lifecycle activities are transparent and tamper-proof.

***

#### **Summary of PublicAI Layers**

| Layer                        | Purpose                                       | Key Components                             |
| ---------------------------- | --------------------------------------------- | ------------------------------------------ |
| DataHub                      | Campaign-based data contribution and voting   | Uploaders, voters, voting-based consensus  |
| Data Hunter                  | Node operations and AI-powered engagement     | Node operators, AI Reply on X              |
| Blockchain & Smart Contracts | Secure, transparent data lifecycle management | BFT consensus, smart contracts, governance |
