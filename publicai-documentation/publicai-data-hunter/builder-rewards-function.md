---
hidden: true
---

# 💰 Builder Rewards Function

**AI Builders** who performed data collection actions receive rewards once the verifications are finished (Except for GPT Chat Data).

The following explains in detail the logic for calculating the number of points earned for each type of Builder task.

## Tweets

**`Data_Hunter_tweet_Reward = function(tweet_quality, builder_level)`**

If you choose to upload tweets using Data Hunter, we will use a professional AI scoring agent to evaluate the quality of the tweets you select.

Additionally, the [level](/broken/pages/pHtSHoMd8pUhXjKI5YZE) also affects the point rewards for Builders. Builders with higher levels receive proportionally increased rewards each time they complete a dataset collection. The specific factors are:

**Beginner: \*1**

**Senior: \*3**

**Master: \*5**

**Note:** the same tweet can only be uploaded once, and the tweet must contain more than 10 words.

## GPT Chat Data

**`Data_Hunter_gpt_Reward = function(chat_quality, builder_level)`**

We will use AI to score the latest five prompts in your uploaded GPT conversations to assess the quality of your prompts. Your points reward will be determined based on the quality and your level.

**Note:** You can earn a maximum of **`1200 * level coefficient`** $PUBLIC Points per day by uploading GPT chat data. Any points beyond this limit will be invalid.

## Check Airdrop Points

You can view the **`$PUBLIC`** points you earn daily through:

* **Answering questions as Validator**
* **Providing contents from X as AI Builder**
* **Completing daily tasks**

&#x20;**`'Reward'`** page displays the total airdrop points.

<figure><img src="../.gitbook/assets/image (43).png" alt="" width="188"><figcaption><p>Reward Page</p></figcaption></figure>
