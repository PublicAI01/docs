# Revenue Driven Token Issue

The most significant feature of PublicAI tokenomics is its revenue-driven token issuance. PublicAI leverages real-world external revenue projects to back the tokens instead of pure-crypto protocol incomes from PVP or inflation to avoid becoming a Ponzi scheme.

<figure><img src="../.gitbook/assets/55 Revenue Issuance Diagram.png" alt=""><figcaption><p>PublicAI Revenue Driven Token Issue Mechanism</p></figcaption></figure>

**AI Clients** include AI firms and related research institutes such as OpenAI and Google, which need AI training data services.&#x20;

The PublicAI platform has four pools: revenue pool, emission pool, stake pool, and reservation pool.

**Revenue Pool** stores AI Clients' payment for the data service including two business schemes: on-demand data service and existing dataset sales. The payment accepts fiats payment(USD, CNY, JAP, SGD, .etc), USDC/USDT stablecoins, and $PUBLIC.&#x20;

**Emission Pool** stores the 50%(0.5B) total supply $PUBLIC tokens for community rewards which consists of two parts: 35% is used to reward the data contributors who upload or verify the data aligns with the data consensus. That part token emission is scheduled by an **Emission Event**. The remaining 15% is used to reward the stakers passively no matter whether the stakers make data contributions or not. The passive reward is 2.5% per year and lasts 6 years.&#x20;

**Emission Event**

The emission process of the emission pool is a business contract or collaboration-wise instead of time-wise compared to most staking protocols. For each emission event, there should be an AI client to sign a business agreement or any new collaboration achieved. Assume the payment budget amount of the collaboration is _**X**_. and the real-time **$PUBLIC** price is _**P**_. The emitted **$PUBLIC** token amount _**Y**_ could be simply calculated as<kbd>_**Y=X/P**_</kbd> . The Y amount $PUBLIC tokens will be fairly rewarded to the community data contributors whose contributions are verified by the [protocol consensus algorithm](bft-data-consensus-algorithm.md).&#x20;

**Stake Pool** contains the **$PUBLIC** token staked by community contributors. The protocol requires them to stake **$PUBLIC** as collateral to back the quality of their contributions to avoid malicious behaviors such as randomly uploading irrelevant or AI-generated data to a data collection campaign or randomly arbitrary voting as validators. The stake amount is dynamically changed due to the task difficulties and the **$PUBLIC** token price. The stake will be slashed for malicious behaviors or if the protocol consensus occurrence is missing.&#x20;

**Reservation Pool** contains the preservation **$PUBLIC** tokens which are bought back by the fiats or stablecoins from the revenue pool at the **Buyback Event**.&#x20;

**Buyback Event** means using the revenue pool's fiats and stablecoins to buy **$PUBLIC** token from Dex or Cex and store it into the reservation pool for future utilities. The event is executed by Public DAO the unique decentralized governance organization to manage the protocol operations.&#x20;





