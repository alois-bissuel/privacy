# AB testing, lift measurement and private unique id count for K-anonymity reporting computation

AB testing is a key feature of the measurement on the web and is widely used to compare from different website layouts to complex machine learning algorithms used in AdTech. There are many ways to run an AB test, each serving a slightly different use-case and each bringing its pros and cons. Below are a few different types of AB tests currently used in the AdTech industry with a quick description of the setup, the use-case they are answering and the set of the metrics we consider.

1. [Opportunity Level AB test](#opportunity-level-ab-testing): The decision to apply policy A or policy B happens at each opportunity to show an ad. This can be used, for example, to decide which ad layout or which product is more appealing to users. Slight variants of this setup exist; what matters here is that the population split is not user-centric: a user could see an ad resulting of policy A once and one resulting of policy B at another time.
2. [User Level AB test](#user-level-ab-testing): On the contrary, many AB tests run by Ad tech actors are user-centric: a user should be consistently part of only one population during the duration of the test. This is the case every time you want to test the efficacy of different "marketing messages/strategies" over time.
3. [Incrementality AB test](#lift-measurement): incrementality AB tests (which allow for lift measurement) is a specific case of user-level AB test where you don't treat (show ads) one population in order to compare the performance with the treated population. If you want to know more about incrementality measurement and why we would want to measure incrementally, please refer to this [blog post](https://medium.com/criteo-labs/incrementality-a-simple-question-commanding-subtle-answers-11b68ce344d8) and [this one](https://github.com/w3c/web-advertising/blob/master/private-lift-measurement-conceptual-overview.md) respectively.

# Table of content

  * [Opportunity-level AB-testing](#opportunity-level-ab-testing)
  * [User-level AB-testing](#user-level-ab-testing)
  * [Lift measurement](#lift-measurement)
  * [Improved k-anonymity computation](#improved-k-anonymity-computation)

## Opportunity Level AB testing

There are 2 major elements necessary to run an AB test:

- the first one is the ability to apply different policies at the opportunity time depending on which population this opportunity falls into.
- the second one is the ability to have a reporting broken down by policy to compare the impacts of different policies.

### At opportunity time

At opportunity time, an opportunity ID can be used by the advertiser to decide what policy to apply for this opportunity and thus have an opportunity level split, as they do today.This opportunity ID is not shared between advertiser and publisher (it could be a fully random Id created by the third party server - or the browser in TURTLEDOVE - not the one used to reconcile bids).

### At reporting time

This opportunity ID is available to the advertiser in the ranked privacy-preserving report described above. Based on this ID available in the report, the advertiser can thus generate a reporting broken down by "policy" since it can know what policy was applied for this particular opportunity ID, hashing it the same way.

Thus, using the reporting already laid out above, the opportunity level AB testing would be fully covered (although you should bear in mind the constraints coming with the ranked privacy-preserving report).

## User-level AB testing

This form of AB testing, that relies on splitting users into several populations and treating each population differently, is at the core of many improvements on the web. As of today, AB testing in web advertising relies mainly on uid hashing that won't be available in the privacy sandbox.

Thus, we must provide with an acceptable privacy-compliant replacement solution that helps make data-driven decisions.

### Our proposal:

- The browser sends at bid time, along with the IG, an *ABTest_ID* between 1 and 16,384 (2^14). This *ABTest_ID* is semi-stable per user (For example, we could imagine that the number for each user would be reset randomly every 2 months).
- The third-party server hashes this *ABTest_ID* differently for each stakeholder (advertisers and publishers), and transform this to an *ABTest_ID_advertiser* between 1 and 32. This ID can be used for AB testing and can be transferred safely ( as an "unprotected variable") in the granular report, as it cannot be used for joining (because the ID for one user is different in the reports produced for the different actors). This enables user-level ABtesting without breaching privacy.

### Value Proposition

The cardinality of the *ABTest_ID_advertiser* is low: this means that the advertiser only has limited options to split the entire pool of users. This also means that an advertiser can run a limited number of tests at the same time.

For example (**In the following example, for the sake of simplicity, the cardinality of the *ABTest_ID_advertiser* is lowered to 10**):

Test 1: 50/50 split

| *ABTest_ID_advertiser* | Population Share | Test 1 Pop |
|----------------------|------------------|------------|
| 0                    | 10%              | Control_1  |
| 1                    | 10%              | Control_1  |
| 2                    | 10%              | Control_1  |
| 3                    | 10%              | Control_1  |
| 4                    | 10%              | Control_1  |
| 5                    | 10%              | Test_1     |
| 6                    | 10%              | Test_1     |
| 7                    | 10%              | Test_1     |
| 8                    | 10%              | Test_1     |
| 9                    | 10%              | Test_1     |

Test 2: 50/50 split

Test 2 split must be orthogonal to Test 1 split for a fair analysis

| *ABTest_ID_advertiser* | Population Share | Test 1 Pop | Test 2 Pop |
|----------------------|------------------|------------|------------|
| 0                    | 10%              | Control_1  | Control_2  |
| 1                    | 10%              | Control_1  | Test_2     |
| 2                    | 10%              | Control_1  | Control_2  |
| 3                    | 10%              | Control_1  | Test_2     |
| 4                    | 10%              | Control_1  | Control_2  |
| 5                    | 10%              | Test_1     | Test_2     |
| 6                    | 10%              | Test_1     | Control_2  |
| 7                    | 10%              | Test_1     | Test_2     |
| 8                    | 10%              | Test_1     | Control_2  |
| 9                    | 10%              | Test_1     | Test_2     |

You can run 2 50/50 split AB tests in parallel. What would happen if the advertiser wanted to run a 90/10 split AB test for the second test (test 2bis)?

Test 2bis: 90/10 split

| *ABTest_ID_advertiser* | Population Share | Test 1 Pop | Test 2bis Pop |
|----------------------|------------------|------------|---------------|
| 0                    | 10%              | Control_1  | ?             |
| 1                    | 10%              | Control_1  | ?             |
| 2                    | 10%              | Control_1  | ?             |
| 3                    | 10%              | Control_1  | ?             |
| 4                    | 10%              | Control_1  | ?             |
| 5                    | 10%              | Test_1     | ?             |
| 6                    | 10%              | Test_1     | ?             |
| 7                    | 10%              | Test_1     | ?             |
| 8                    | 10%              | Test_1     | ?             |
| 9                    | 10%              | Test_1     | ?             |

A 90/10 split, orthogonal to the first test already running is impossible in these conditions. While two 50/50 splits at the same time were possible, a 50/50 and 90/10 were not: tradeoffs exist between the number test you can run at the same time and the population ratios available.

Currently, companies like AdTech providers run tens or hundreds of AB tests simultaneously. This AB test framework would thus be extremely limiting and would require drastic changes in the way they drive their operations. On the other hand, the low cardinality of *ABTest_ID_advertiser* allows it to remain an unprotected variable, meaning that running an AB test won't impact the reporting "quality" and that **advertiser won't be forced to adapt the reporting dimensions depending on the AB tests it is running**.

### Privacy Considerations

As already mentioned, the low cardinality of the *ABTest_ID_advertiser* makes it a limited threat to user privacy despite being treated as an "unprotected variable".

Furthermore, the *ABTest_ID_advertiser* for one user is specific to each advertiser (the same user would have potentially different *ABTest_ID_advertiser* for different advertisers). This means that colluding advertisers would not be able to use this variable to join their reporting.

### Improved K-anonymity computation

This *ABTest_ID* is also used by the third-party server to estimate the number of unique UIDs without having ever access to the UIDs themselves (remember that the third-party server must be able to get the information to apply the K-anonymity filter for the granular reporting). Indeed, we can use the number of distinct ABTest_IDs as a proxy for the number of unique UIDs. For example, if k = 3, the server verifies that there are at least 3 rows for different ABTest_IDs. While it is theoretically possible that two different users have the same *ABTest_ID*, this is unlikely enough (if k remains small and the cardinality of ABTest_ID remains high) so the impact on the report should be very limited.

**This approach means that no unique UIDs are passed to the third-party server**, lowering the amount of trust in them required.  If 2^14 of ABTest_IDs seems too high, consider that with billions of unique UIDs this would reduce the number of UIDs per semi-stable group to hundreds of thousands. These groups would clearly be too large to be a credible threat to user privacy.

## Lift Measurement

Lift measurement requires a form of AB test where a population is not exposed to advertising.

For this part, we built upon the 2 proposals for private lift measurement by Facebook, [here]( https://github.com/w3c/web-advertising/blob/master/private-lift-measurement-first-party.md) and [here]( https://github.com/w3c/web-advertising/blob/master/private-lift-measurement-third-party.md). These APIs are designed to measure the lift of a particular domain and/or a particular ad network. We wish to propose a more flexible framework that would not only answer the use case detailed in these articles but also help advertisers to measure the lift on other dimensions. For example, advertisers must be able to measure the lift brought by different DSPs/ marketing channels and compare them in order to allocate their budget appropriately.

### Initial Proposal

#### Proposal

- Allow a browser to consistently assign users to either a test or control group for an advertiser/redirection domain X a set of dimensions of the choice of the advertiser. Test/Control assignment depends on the redirection domain and a custom hash function.
- Allow the browser to enforce that decision by only showing the ad to people in the test group, suppressing the ad in the control group (show another ad instead).
- The third-party server provides an aggregated reporting, including the sales of the non-treated population.

#### Displaying Ads
- The advertiser (the trusted third party) provides 2 ads: an ad to show to people in the “test group”, and a “fallback ad” to show to people in the control group. The fallback ad can be either a PSA or PSA-like ad or the second choice in the auction).
The first ad contains the ratio *testGroupPercentage* of users in the treated population (who see ads), a hash function and an optional SCOPE, based on the contextual signal.
- The first time the browser needs to make a test/control decision about ads being shown by a given advertiser, it just needs to generate (and store) a random identifier: *advertiser_specific_random_id*. This same identifier will be retrieved by the browser from storage and used on all subsequent test/control decisions made in that browser for ads served by the same advertiser.
- Decision:
	`is_test = IF(contextual_signal IN SCOPE custom_hash_function(advertiser_specific_random_id, IG) % 100 < testGroupPercentage) ELSE TRUE END`
- When such a decision occurs for an advertiser, we say that an opportunity occurred, whether the ad was eventually printed or not.

#### Reporting Conversions

We use here a similar approach to the “Private Click Measurement” and “Conversion Measurement API”.

- The advertiser reports the conversions in a ./well-known location with the various hash_function_id as metadata.
- Browser accesses conversions (./well-known)
- The browser can use the hash functions and the scope in the conversion to get the user's population for each campaign/channel the user was eligible to and sends out a report for each for the conversion to the third party server if the user had one or more opportunities in the defined scope:
	- For each conversion onsite the browser generates N reports for the N hash functions the user had an opportunity for.

**Example**: The user is eligible to 3 IGs, each one with a different split/hash functions (and no defined scope), with the following hash_function_id: “abc”, “def” and “fgh” and converted once on *shop.example*.

He received 2 opportunities for IG with hash_function_id “abc” for which he is in the treatment group (thus 2 displays), 1 opportunity for “def” where is in the control group (thus 0 display) and 0 opportunities for “fgh”.

The generated “reports” would look like this below:

|        Key       |     Value    |
|:----------------:|:------------:|
| reporting-domain | shop.example |
| hash_function_id | abc          |
| isTreated        | 1            |
| Conversion       | 1            |
| Conversion Value | XX           |

|        Key       |     Value    |
|:----------------:|:------------:|
| reporting-domain | shop.example |
| hash_function_id | def          |
| isTreated        | 0            |
| Conversion       | 1            |
| Conversion Value | XX           |

This implies however a change from normal PCM: the browser needs to store more than just clicks, now it must store opportunities to see the ad. We could user something resembling the "trail store" described in [SPURFOWL](https://github.com/AdRoll/privacy/blob/main/SPURFOWL.md), but with the opportunities on top of the other events.

The trusted third-party server aggregates the results and creates a lift aggregated report for each hash function (**With K-anonymity treatment**), adding another report (a fourth one) to the list.

|        Key       |     Value    |
|:----------------:|:------------:|
| reporting-domain | shop.example |
| hash_function_id | abc          |
| isTreated        | 1            |
| Nbr Users        | 33,000       |
| Opportunities    | 100,289      |
| Conversion       | 123          |
| Conversion Value | 1340         |

And a similar report for isTreated = 0.

#### Value Proposition

##### hash function

The *advertiser_specific_random_id* allows the advertiser for the required measurement of the lift brought by its marketing effort as a whole but it is not enough. Indeed, to provide a workable performance framework, the privacy sandbox must allow advertisers to compare the performance of their various campaigns/marketing channels, pilots these channels according to the performance they bring and provide some flexibility in the dimensions you can assess.

Thus we don’t want only to measure the lift/incrementality at the redirection site level (or reporting domain level) but also to measure the impact of each individual campaign/Interest group among other dimensions. The hash function and the scope make sure that the user isn’t consistently assigned to one population or the other for a redirection domain but for a redirection domain X custom dimensions defined by the advertiser using the interest group and the contextual signals available.

Thanks to this custom hash, a user can be in the treatment group for IG_1 of *shop.example* and in the control group for IG_2 of the same site (*shop.example*).

##### Scope

The scope would be useful for an advertiser to test the incrementality of a publisher. Using the contextual signal, the hash function would set the user as part of the test or the control population, but only for the publisher domain, defined as "in scope".

##### Head-to-Head performance comparison

The hash enables an advertiser to compare the performance of two competing marketing channels. For example, *shop.example* wants to compare the lift/incrementality brought by the two remarketers it is working with: AdTech1 and AdTech2.

AdTech1 and AdTech2 would use different functions, thus creating a different split. Each can thus measure the incrementality is bringing to *shop.example* independently of the other.

##### Conversions for the controlled group:

Having access to the conversions in both groups (treatment and control) only for users who had opportunities (displays for one group and "shadow displays" for the other respectively) is extremely beneficial both for minimizing the measurement noise and for the ability to run lift measurement throughout the funnel with a similar framework!

Indeed this allows for measuring the impact of ads taking into accounts only the users who have effectively been impacted (have seen at least one ad in scope).

#### Privacy Considerations

In order to prevent publishers to track and recognize users across sessions even if he deleted his cookies, the *advertiser_specific_random_id*s would be reset as well.

Furthermore, this *advertiser_specific_random_id* presents a high risk of being utilized for cross-site tracking. Even if the identifier itself is not accessible to application code, any kind of leakage of information about this identifier could potentially be used for cross-site tracking.

This implies that it must be **technically impossible** for the publisher to know which ad the person saw (the primary ad or the fallback ad).

The implications of this are discussed [here](https://github.com/w3c/web-advertising/blob/master/private-lift-measurement-third-party.md). While showing ads in a fenced frame and controlling what the publisher can have access to has been discussed and is covered the privacy sandbox, it still raises questions when it comes to pure contextual advertising outside of the privacy sandbox.

Such an attack would require an extreme level of coordination and collusion between the supply side and the advertiser side but would be workable. This needs to be discussed further into details to come up with countermeasures.

#### Other considerations

In order to try to provide consistent test/control status for the same person across multiple devices they own, the browser can sync the *advertiser_specific_random_id* from one device to another.

In order to measure conversion events where the ad-opportunity / ad-impression was on one browser, and the conversion was on another, not only the *advertiser_specific_random_id* but also the locally stored “ad-opportunities requesting attribution” could be synced across devices belonging to the same user.

### Extended Proposal: Have multiple hash functions

Allowing multiple hash functions, each defining a population the user is a part of would enable to have multiple layers of lift measurement running at the same time and thus makes this measurement framework extremely flexible.

For example, if the advertiser passes two hash functions:

- The first layer could allow (for example) for a general lift measurement at the domain level and allow the advertiser to have a broad overview of the performance brought by its marketing efforts.
- The second layer, orthogonal to the first one could allow for campaign level lift measurement and could be used for budget allocation decisions across campaigns and "piloting".

With multiple hash functions, at ad serving time, the decision would be taken by the browser by computing the population for each function and then decide whether the ad should be displayed based on the intersection of the various outputs:

Decision example with 2 functions:

- `isTest1 = IF(contextual_signal IN SCOPE hash_function_1(*advertiser_specific_random_id*, IG) % 100 < *testGroupPercentage*1) ELSE TRUE END`
- `isTest2 = IF(contextual_signal IN SCOPE hash_function_2(*advertiser_specific_random_id*, IG) % 100 < *testGroupPercentage*2) ELSE TRUE END`
- `showAd = isTest1 && isTest2`

The reports including hash_function_id as a dimension, the user and his/her conversions if any would be "included" in the reporting for each hash function.

#### Example

Alice (whom *advertiser_specific_random_id* for *shop.example* is *alice_shop_example_id*) is assigned to one IG (named *IG*) for advertiser *shop.example*. The advertiser wants to measure independently the lift brought by:

1. Its entire marketing effort
2. One publisher: publisher.com

In the web bundle of *IG*, advertiser adds 2 hash functions and 2 related *testGroupPercentage*. Only the second has a scope.

1. `hash_function_1, x, SCOPE_1 = NULL`
2. `hash _function_2, y, SCOPE_2 = publisher.com`

In the example, both functions would be added to all IGs since the measurement the advertiser wishes to run in cross interest-group.

The second "assignment function" displays the ad to all users outside of publisher.com. On publisher.com, the hashing function would allocate the users to the two populations randomly.

Alice visits publisher.com and gets an opportunity for *shop.example* (*shop.example* wins the auction). The browser now gets to decide if it is to print the ad for *shop.example*.

`showAd = ((hash_function_1(alice_shop_example_id) % 100 < x) && (hash_function_2(alice_shop_example_id) % 100 < y)))`

Let's say for the sake of the example that the first term is True but the second term is False. The ad is thus not shown to Alice and another is put in its place.

Alice visits another publisher news.com, which is "out of scope".

`showAd = ((hash_function_1(alice_shop_example_id) % 100 < x) && True = ((hash_function_1(alice_shop_example_id) % 100 < x)`

The first term is the same as on publisher.com (True for alice_shop_example_id). The second term on the other hand is True for all users on this publisher. Thus, this time the ad is shown.

Then Alice comes back to *shop.example* and converts onsite.

The browser generates 2 "reports", sent out to the trusted third party:

|        Key       |      Value      |
|:----------------:|:---------------:|
| reporting-domain | shop.example    |
| hash_function_id | hash_function_1 |
| isTreated        | 1               |
| Conversion       | 1               |
| Conversion Value | X               |

|        Key       |      Value      |
|:----------------:|:---------------:|
| reporting-domain | shop.example    |
| hash_function_id | hash_function_2 |
| isTreated        | 0               |
| Conversion       | 1               |
| Conversion Value | X               |

Had Alice not browsed publisher.com, the second report would not have been generated.

Thus the lift measured thanks to the second hash and scope is the difference in performance between the users who had at least one opportunity on publisher.com and were treated and those who had at least one opportunity on publisher.com and were not treated.

The advertiser receives the aggregated reports:

| reporting-domain | hash_function_id | isTreated | Nbr Users | Opportunities | Conversions | Conversion Value |
|:----------------:|:----------------:|:---------:|:---------:|:-------------:|:-----------:|:----------------:|
| shop.example     | hash_function_1  | 1         | 123,823   | 15,201,456    | 1,456       | 9,463            |
| shop.example     | hash_function_1  | 0         | 123,759   | 15,202,439    | 1,047       | 4,479            |
| shop.example     | hash_function_2  | 1         | 12,436    | 1,302,732     | 145         | 879              |
| shop.example     | hash_function_2  | 0         | 12,435    | 1,302,498     | 142         | 856              |

The advertiser can use these aggregated reports to compute the lift measures it needed.

### Proposal 2

This relies on a completely different idea.
Here, we propose another semi-stable ID, *LIFT_ID* defined by the browser. As for the *ABTest_ID* we could imagine, for example, that the *LIFT_ID*  for each user would be reset randomly every 2-3 months.

The *LIFT_ID* would have a low cardinality (~10) but would be common to all actors (a user would be in the same *LIFT_ID* bucket for all advertisers). For this reason, this variable would be considered protected in the ranked-privacy preserving granular report.

#### Value Proposal

The use of the *LIFT_ID* is quite similar to the *ABTest_ID_advertiser*. The advertiser would use the buckets to treat some of them and not treat the others and measure the difference in performance between the two.

**Pros**:

The buckets being shared by multiple actors, it is convenient to precisely split the population, agnostically of the actor. For example, if *shop.example* want to run a head-to-head comparison of two marketing providers, he can use this ID to split the user pool fairly.

**Cons**:

This method doesn't give access to the pool of users in the control population (untreated) who would have been treated, had they been in the treatment group (users who had an opportunity) and their related onsite sales. The only way to compare both groups is to consider the entire pool of users in the Interest Group. That means that the pool of users is bigger and the results are noisier. This is particularly true for non-retargeting use cases.

In a pure contextual example: users are not known to the advertiser previously to showing an ad.
As the *LIFT_ID* is always available, when a user comes to the site and maybe converts, the advertiser can know if this user belongs to the treatment or the control group. However, there is no way to distinguish users who have been exposed (or at least had an opportunity for users in the control group) from those who have not. Thus, you must include all users in your comparisons.

In a retargeting use case, you can consider in the pool of users only the web users who belong to one of your interest groups, limiting it a lot already but in the contextual use case, you must consider all web users, although you know that you impacted a tiny share of them.

The very fact that the pool of users that take into account is different from one use case to the other skews the performance comparison.
