# platon-random-wasm

Using economic models to generate decentralized random numbers in the public domain.

## Solutions

The process of generating random numbers is divided into three stages:

**The first submission stage: collect valid sha3(s)**

Those who want to participate in generating random numbers, within a specified time period (for example, 10 blocks, about 10 seconds) send m contract deployment chain main coins to contract C as collateral and attach the result of sha3(s) , S is the secret number chosen by the participant.

**The second stage of disclosure: collecting valid s**

After the first stage, the person who successfully submits sha3(s) must send a transaction with the secret number s to contract C before the random number generates the target block height. Contract C performs sha3 operation on s and compares the result with the previously submitted data.
The s whose comparison results are equal will be stored in the seed set and participate in the generation of random numbers.

**The third stage of dividends: calculating random numbers, refunding deposits and bonuses**

1. After successfully collecting all secret numbers, contract C calculates a random number according to the random number generation function f(s1, s2, ... sn), the calculation result will be written into the storage of C, and those who need random numbers can pass the contract Call to get a random number.
2. Contract C returns the deposit of the first stage to the participants, and distributes the profits in equal shares as additional rewards to all participants. Revenue refers to the cost paid by the random number demander.

In order to ensure that the random number is not operated and take into account efficiency and safety, the following constraints must be met:
1. In the first stage, when more than two identical sha3(s) are submitted in order, only the first one will be accepted.
2. In the first stage, the minimum threshold for the number of participants is set. If enough sha3(s) is not collected in this time period, the random number generation in this round fails.
3. Participants who submit sha3(s) and are accepted must disclose the s in the second stage.
    1. When the submitter fails to disclose s in the second stage, the deposit provided in the first stage is confiscated and no return is provided.
    2. When one or more s are not disclosed in the second stage, this round of random number generation fails. The confiscated margin is divided into equal parts and distributed to other participants who disclosed s in the second phase. The cost paid by the random number demander shall be refunded.

## Specification

### Contract method specification

#### NewCampaign

``` C++
ACTION u128 NewCampaign(uint64_t burn_num, const u128 &deposit, uint16_t commit_balkline, uint16_t commit_deadline);
```

**Brief:** Create a new round of activities to generate random numbers

**Param:** burn_num The target of random number generation is faster

**Param:** deposit Deposit submitted by the participant

**Param:** commit_balkline The distance from the beginning of the commit phase to the target block

**Param:** commit_deadline The distance from the end of the commit phase to the target block

**Return:** Random number generation activity ID

#### Follow

``` C++
 ACTION void Follow(u128 campaign_id);
```

**Brief:** The random number demander can choose to follow a certain round of existing random number generation activities

**Param:** campaign_id An existing random number generation campaign ID

#### Commit

``` C++
ACTION void Commit(u128 campaign_id, const bytes32 &hs);
```

**Brief:** Participants submit the sha3(s) phase. To submit a random number, a deposit must be sent to the contract, and the random number must be submitted during the random number submission window.

**Param:** campaign_id An existing random number generation campaign ID

**Param:** the sha3 result value of the hs secret number

#### Reveal

``` C++
ACTION void Reveal(u128 campaign_id, u128 s);
```

**Brief:** Disclosure of the random number stage. After the submission phase is over, it enters the disclosure phase. The calculation result of the disclosure value sha3 must be equal to the value of the submission phase. It does not mean that the margin will be confiscated and there will be no dividends. Only valid values ​​that pass verification will participate in the generation of random numbers.

**Param:** campaign_id An existing random number generation campaign ID

**Param:** s is the secret number involved in generating random numbers

#### GetRandom

``` C++
CONST u128 GetRandom(u128 campaign_id);
```

**Brief:** Get a random number. Anyone can get the random number of the round of activities after the random number target block. Only when all submitters disclose secret numbers will this round of activities be effective and generate random numbers.

**Param:** campaign_id An existing random number generation campaign ID

**Return:** Random number generated in this round of activity

#### GetMyBounty

``` C++
ACTION void GetMyBounty(u128 campaign_id);
```

**Brief:** Get bonuses and deposits. After the random number generates the target block, the random number submitter can recover the deposit and income.
When the random number generation activity is successful, the reward provided by the demander will be divided equally, and the deposit will be returned.
When the random number generation activity fails, the deposits of the participants who have not disclosed the secret numbers are equally divided, and the deposits are returned. No one discloses the random numbers, and all participants get their deposits back.

**Param:** campaign_id The campaign ID that generates a random number

#### RefundBounty

``` C++
ACTION void RefundBounty(u128 campaign_id);
```

**Brief:** Refund the bonus. If the random number generation fails, the random number demander returns the reward it provides through this function.

**Param:** ampaign_id The activity ID that failed to generate a random number

### Contract event specification

#### LogCampaignAdded

```c++
PLATON_EVENT2(LogCampaignAdded, u128 campaign_id, platon::Address from, uint64_t burn_num, u128 deposit, uint16_t commit_balkline, uint16_t commit_deadline, platon::u128 bountypot);
```

**Brief:** Create a new round of random number generation event

**Param:** campaign_id Random number generation campaign ID

**Param:** from the event creator is the random number demander

**Param:** burn_num The target of random number generation is faster

**Param:** deposit Deposit submitted by the participant

**Param:** commit_balkline The distance from the beginning of the commit phase to the target block

**Param:** commit_deadline The distance from the end of the commit phase to the target block

**Param:** Bountypot The cost paid by the demander

#### LogFollow

``` C++
PLATON_EVENT2(LogFollow, u128 campaign_id, platon::Address from, u128 bountypot);
```

**Brief:** Follow the existing random number to generate an event

**Param:** campaign_id Random number generation campaign ID

**Param:** from random number demander

**Param:** Bountypot The cost paid by the demander

#### LogCommit

``` C++
PLATON_EVENT2(LogCommit, u128 campaign_id, platon::Address from, bytes32 hs);
```

**Brief:** Submit secret number sha3 value event

**Param:** campaign_id Random number generation campaign ID

**Param:** from random number participant

**Param:** the sha3 result value of the hs secret number

#### LogCommit

``` C++
PLATON_EVENT2(LogReveal, u128 campaign_id, platon::Address from, u128 s);
```

**Brief:** Disclosure of secret number s value event

**Param:** campaign_id Random number generation campaign ID

**Param:** from random number participant

**Param:** s is the secret number involved in generating random numbers