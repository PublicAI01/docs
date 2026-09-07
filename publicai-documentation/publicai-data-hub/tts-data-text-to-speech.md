---
hidden: true
---

# 🗣️ TTS Data (Text-to-speech)

**TTS data** is crucial for large language models and multimodal AI systems, enhancing speech synthesis and cross-modal capabilities. However, current speech models often underperform due to limitations in TTS data, including insufficient scale, diversity, and quality, especially for **non-English languages**. Despite these challenges, high-quality TTS data provides valuable speech-text alignment information, aiding in complex language learning. It's essential for pretraining tasks and developing expressive AI assistants. Improving TTS datasets remains a key priority to advance speech technology and enhance human-machine interactions.

**The TTS data collection, validation, annotation, and development functions of PublicAI Data Hub are now online.** The following steps will help you understand how to participate in the PublicAI TTS data ecosystem.

## TTS Builder: Record audio according to the prompts

Login to [PublicAI Data Hub](https://beta.publicai.io/). On the **Builder** page, You will see **"Audio Datasets"** cards in different languages. Open the card for the language you're interested in, and the Data Hub will automatically generate ten voice recording tasks.

<figure><img src="../.gitbook/assets/image (50).png" alt=""><figcaption><p>Builder Page</p></figcaption></figure>

After clicking to enter the card for the corresponding language, you need to read out the system-generated **content** according to the prompted **tone**. Click the recording button to start recording, and then submit it.

<figure><img src="../.gitbook/assets/image (51).png" alt=""><figcaption><p>Recording Page</p></figcaption></figure>

**Special Reminder:**

1. PublicAI system uses professional AI voice models to recognize what you say. If it differs significantly from the given content, you won't be able to submit it. This serves as an initial screening to ensure data quality remains at a certain standard.
2. You can submit up to 20 audio recordings per day.

**TTS builder's reward formula:**&#x20;

`Reward = [100 * (Voice Upload Quality)^2 * Level Coefficient]`

The results for recordings uploaded by Builders will only appear after voting by Validators. If the majority agrees that the recording matches the given content and tone, the Builder will receive the full reward.

If only one aspect (either content or tone) passes the consensus, the Builder will receive half the points.

If the recording fails to achieve consensus from the majority, the Builder will not receive any point rewards.

## TTS Validator: Evaluate audio: content and tone

On the Validator page, you'll see "Train Audio Datasets" cards. Open the card for the corresponding language to become a TTS Validator.

<figure><img src="../.gitbook/assets/image (52).png" alt=""><figcaption><p>Validator Page</p></figcaption></figure>

After clicking to enter the TTS Datasets for the corresponding language, validators can listen to the recordings contributed by builders. Based on their intuition, validators should **input the content they hear** and **select the tone** that matches the recording.

<figure><img src="../.gitbook/assets/image (53).png" alt=""><figcaption><p>Validation Page</p></figcaption></figure>

**TTS builder's reward formula:**&#x20;

`Reward = [Total historical accuracy * (Current accuracy / 0.5 + 1) * User level * Base point reward for answering * Ranking coefficient]`

\*The ranking coefficient refers to the order in which validators verify the recording. Each recording requires 81 validators to vote. The 1st to 20th validators receive a coefficient of 1.2, the 21st to 40th receive a coefficient of 0.7, and the 41st to 81st receive a coefficient of 0.2.
