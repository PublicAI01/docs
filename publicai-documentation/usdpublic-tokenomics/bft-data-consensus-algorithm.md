# BFT Data Consensus Algorithm

<figure><img src="../.gitbook/assets/Group 1312317838 (2).png" alt="" width="563"><figcaption><p>BFT Data Consensus</p></figcaption></figure>

BFT data consensus algorithm is the key mechanism for PublicAI to guarantee the quality of data which consists of  two phases:

**AI Screening Phase**

Once the uploader uploads the data to the platform, our corresponding AI agents will filter it as the first step to mitigate the cost of human verification. Those AI agents are various in terms of data.&#x20;

**Human Voting Phase**

After the uploaded data is accepted by AI agents, it will enter the human verification phase. Consensus is the same number of votes in the same direction. Data sample quality is binary: if a data sample is partially good then it's considered overall flawed.&#x20;

We have three roles in the protocol to verify: **scouts, guards, and judges** with three hyper-parameters for them respectively as consensus thresholds.&#x20;

<figure><img src="../.gitbook/assets/image (4).png" alt="" width="563"><figcaption><p>Concensus Reach Scenarios</p></figcaption></figure>

Take an example from the above graph,  once a data sample is accepted by the data consensus algorithm there should be at least either 50 votes from scouts, 5 votes from guards, or 1 vote from a judge. &#x20;
