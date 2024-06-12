| PIP               | Title                           | Description          | Author                        | Discussion | Status | Type                                     | Date                  |
|-------------------|---------------------------------|----------------------|-------------------------------|------------|--------|------------------------------------------|-----------------------|
| 39 | Validator Admissions into PoS Network  | Proposes Validator Admission Framework | Jackson Lewis, Tanisha Katara, Parvez Shaikh, Vasanti Rode | Forum  | Draft | Contracts | 2023-12-05

### Abstract

[PIP-4](https://forum.polygon.technology/t/pip-4-validator-performance-management/9956) Part C, outlines the path of Validator Admissions into the network. This proposal outlines an admissions framework that includes a set of objective parameters to determine a prospective validator's eligibility to validate the PoS network.

### Motivation

A healthy network requires healthy validators/operators. The aim of a validator admission process is to:
1) ensure a competent set of validators on PoS with greater transparency and community input on validator admissions, and
2) ensure network security, reliability and uptime.

Having a quantitative, objective process in place will ensure that those who are becoming part of the network are experienced and able to participate at a high rate of adherence to on-chain operations.

A large, mature and widely used network like the PoS chain, requires quality validators who uphold reliability and performance to meet the demands of the industry in which they operate.

The proposed admissions process assists in providing the network with a framework to evaluate potential validators who request to join the PoS network. This process scores and ranks applicants based on the evaluation criteria described in the specification.

### Rationale

In accordance with PIP-4, a scoring system that will rank validator applicants is proposed, allowing only the highest-scoring candidates to be selected for onboarding to the network.

To define the scoring system, identified below are characteristics of validators that correlate with high performance; utilizing a number of heuristics based on the correlation matrix.

The scoring is composed of the three following factors:

1. **Stake**

MATIC stake to provide enhanced network security.

2. **Experience**

Number of years operating a validating node on a PoS network.

3. **Expertise**

Validator operators score on an assessment test.

The correlation matrix below shows a moderately positive correlation between PoS validator experience and performance. In other words, validators with more experience tend to perform better.

Based on these findings, adjustments were made to the scoring logic used to evaluate the prospective validators. These adjustments ensured a fair and accurate performance assessment, considering their experience and expertise.

It is important to note that while experience was found to have a high correlation with performance, stake and expertise also played significant roles in determining a validator's effectiveness. Therefore, a holistic approach was taken to consider multiple factors in the scoring logic adjustments.

Using the insights gained from this survey, the evaluation process was improved, and informed decisions regarding the validators' contributions and rewards can continue to be made.

![|624x221](https://lh7-eu.googleusercontent.com/docsz/AD_4nXe40GRphZCkLCGLph7wiKkoaabo2gIT-w-xaxo-5kXBSJ9HADRjvp9QrSu6jTkBpftc0l3VDaSbotwAvzAYVd681RXHxzmP-3G1AcN1TjJ0UCN-egkcwbRyS5mM-4HXIBshjRnlOhzwE1I4UKg_IbVASctj?key=WpqSAzkRRGAd_g58htzluQ)

The Pearson correlation coefficient was used to quantify the relationship between performance and experience for Polygon PoS’s current validators, which measures the linear association between two variables. It has a value between -1 and 1 where:

-1 indicates a perfectly negative linear correlation between two variables

0 indicates no linear correlation between the two variables

1 indicates a perfectly positive linear correlation between two variables

The further away the correlation coefficient is from zero, the stronger the relationship between the two variables.

#### Results:

Thus far, 12 entities have been recently onboarded to the network through the new admissions process, with zero breakages or bugs in the test phase.

To illustrate the effectiveness of this process, below is data referring to the amount of times these validators fell into Grace Period 1 in their first 2100 checkpoints after joining the network.

**The [PIP4 performance management process](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-04.md) outlines that if a validator falls below a 95% performance in signing checkpoints, they move into Grace Period 1. This Grace Period is a 700 checkpoint window in which they have to regain 95% adherence. If they once again fail to meet the 95% adherence requirement, they fall into Grace Period 2, another 700 checkpoint period. If they then again fail to meet this 95% requirement they then receive Final Notice and are removed from the network.*

Regaining adherence moves them straight back into healthy status.

The following table shows the validators who were recently onboarded through this Admissions Process and the corresponding times they fell into Grace Period 1 in their first 2100 checkpoints in the network. Only 3 of these 12 validators have entered the grace period, reflecting a healthy admission process.

|Validator ID|Name|# of times in GP1|Currently in active set|
| --- | --- | --- | --- |
|170|Sensei Node|1|Yes|
|156|Nexon|0|No|
|163|Upbit|0|Yes|
|161|Northstake|2|Yes|
|152|Zee Prime|1|No|
|162|P2P Network|0|Yes|
|164|Validatrium|0|Yes|
|165|Simply Staking|0|Yes|
|159|Mantra DAO|0|Yes|
|160|NEOPIN|0|Yes|
|167|Ryabina|0|Yes|

Notes:

[1. Update: PoS Validator Admissions](https://forum.polygon.technology/t/update-pos-validator-admissions/12344)

[2. PoS Admissions Update](https://forum.polygon.technology/t/pos-admissions-update/11348)

#### Impact:

All of these validators have performed very well since joining. Those that have left the active set have left for reasons other than performance. A direct correlation has been shown between these validators' scores and their ability to perform validation services in the network.

### Specification

#### Scoring Logic

The final scoring logic is outlined below:

1. Stake: 45%
2. Experience: 25%
3. Expertise: 30%

The expertise section is categorized into 3 parts:

* Part 1: Technical Assessment - 9 Questions +1 for every correct response. No negative scoring.
* Part 2: Governance Assessment - 5 Questions +1 for every correct response. No negative scoring.
* Part 3: Outsourced Operations +1 if none of the validator operations are outsourced. No negative scoring.

To avoid foul play, the following measures have been introduced:

1. All assessment questions will be randomized and changed periodically
2. All assessment questions will be timed
3. All assessment answer options will be randomised

Highly scored validators across these 3 areas can be considered eligible. An eligible score does not guarantee admission. Admission is subject to verification.

### Backwards compatibility

This proposal does not introduce any breaking changes and is therefore backwards compatible.

### Security Considerations

The admissions process takes into account an applicant's hosting region, on-chain activity in other protocols and general security practices around their hardware (i.e., how they operate their own internal systems).

The implementation of this PIP proves no direct security risk to the protocol.

### Copyright

All copyrights and related rights in this work are waived under [CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).

### Appendix

The scoring logic and weights were an extensive exercise in decision-making, impactfully combining data and observation.

**Activity 1:** A survey of high-performing validators measured the correlation between stake, experience, expertise, and performance.
**Activity 2:** A community study to mathematically measure and define the three parameters of evaluation: Experience, Expertise and Stake.

Experience: > 3 years as a PoS validator considered as high experience
Expertise: High percentile in the validator assessment quiz (designed to test Polygon PoS understanding)

Stake: Meeting the minimum staking requirement of the network to ensure economic security
