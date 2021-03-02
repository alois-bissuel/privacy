# 3 reports covering all "usual" measurement use-cases while preserving full privacy

## Table of content

  * [Aggregated report](#aggregated-report)
  * [Report on ads served](#report-on-ads-served)
  * [Ranked-privacy preserving granular report](#ranked-privacy-preserving-granular-report)
  * [Optional granular report on lost opportunities](#optional-granular-report-on-lost-opportunities)
  * [Regarding conversion attribution](#regarding-conversion-attribution)

[Appendix](#appendix)

## Aggregated report

This report enables near-real-time management for advertisers and publishers. It relies on differential privacy mechanisms.

Variables such as interest groups, number of displays, number of viewed ads, number of clicks, prices, and "creative IDs" can be exposed here in a privacy-preserving way while keeping the near real-time property which is critical for ad tech players.

However, in order for this report to remain relevant for certain use-cases such as billing, the noise (the differential privacy epsilon) must be restrained. Otherwise, yet another report meeting another set of requirements would be needed for these use-cases.

## Report on ads served

Being able to fully meet the ad quality use case is paramount to avoid not only bad quality ads but also to protect the user from malware advertising. Ad quality is different from many other use cases as it cannot be dealt with an "on average policy". Each case must be handled independently. In order to ensure that all rendered ads were compliant with the publisher ad quality guidelines and, even more important, user safety, the publisher receives **all** web bundles with the click-through URL domain, creative id and the advertiser name. Although the other two types of reports are meant to offer flexibility and completeness while respecting privacy standards, they may not be enough to handle publisher Ad Quality. For this use case, a separate "report on ads served" is necessary.

Besides the complete list, this report does not pass much information and some delay (~24 hours) remains acceptable.

## Ranked-privacy preserving granular report

This report is granular, meaning that each row maps to a display and the label (click/sale) is associated with the exact display for which it occurred. This report can, for example, be used for performance optimization, bug detection on the advertiser side and fraud detection on both sides. While the billing is already covered by the differential private report (Cf. above), this report can help actors get an exact report for this use-case.

It aims to offer advertisers and publishers a trade-off between the granularity and KPIs accuracy to fulfill their needs while preserving the identity of each user.

To do so, the trusted third party server provides a granular report to the advertiser with k-anonymity on a subset of reported variables (that we call here 'protected variables') with some delay. All the variables that could be used to tie the protected variables together (a.k.a. personal identifiable information (PII)) is obfuscated following the process described below. However, non-protected variables (variables only accessible by one side), such as the click log (is not available in the publisher report), is always recorded at the display level. Therefore, we are preventing any unique join key to appear in the reports to ensure privacy whilst passing as much information as possible to the advertiser and the publisher.

Each actor is limited in the number of queries he can run over each individual raw data point. Without this limit, the ranking of the available dimensions and the K-anonymity wouldn't protect anything. Each display (one row in the raw data) can only be queried once by each actor.

### Non-protected variables

Many variables are relevant or accessible to one player only. For example, ad labels (click ID, opportunity ID, etc.), ABTest ID (see ABTest section below), or the interest group for the advertiser. These variables should be hidden to the other players and do not need other specific attention in this report, they do not represent any risk with regards to the user privacy and can be reported as is at the display granularity.

Some of these variables pose no threat to user privacy and are crucial for advertisers to run performing campaigns. Thus, the ability to get access to them freely (without getting into the granularity vs threshold tradeoff) is extremely beneficial.

### Protected variables

On the other hand, some variables are relevant to both the advertiser and the publisher and represent a privacy risk if exposed as is in the reports. For example:

- Contextual data
- The price of displaying the ad
- The winning DSP / the click-through domain.

Contextual data are very valuable but not all use cases / not all vendors value the same type of contextual data. The report allows the advertiser and the publisher to rank raw contextual variables (e.g. display size) or transformed contextual variables (e.g. truncated URL) by importance according to them, in order to have as much valuable information as possible whilst still preventing colluding publishers and advertisers to join a user with an interest group.

The report relies on k-anonymity, meaning that a feature is available only if at least k similar records (at the UID level) are available (e.g. for a truncated URL to be revealed, you need at least k requests on this truncated URL and the same higher ranked features' modalities). Please refer below for detailed examples of how this system works.

This constraint makes full URLs, with potential PII encoded therein, not usable in such report. The need for providing the transformation to remove these privacy-breaching parts of the URL is thus transferred to the advertiser and publisher (and not the reporter) incentivized by the k-anonymity constraint.

The ranking (provided by the advertiser or publisher) allows the reporter to know which dimension should be sacrificed in case k-anonymity is not respected for a group of reported rows. For k-anonymity to work, continuous variables such as conversion amount (currency) need to be pre-processed either by bucketing them.

### Examples of ranked privacy-preserving report

**The raw data**:

Here we have 9 displays (each with an individual opportunity_ID), to 9 different users, with 4 publisher features, the publisher_UID for each user, the domain, the subdomain, the display size, and a label. The opportunity ID is not shared between the advertiser and publisher.

In this example, we consider **k=2**.

The raw data is the following:

| opportunity_ID | publisher_UID | Domain | Subdomain | Size | Label |
|----------------|---------------|--------|-----------|------|-------|
| abc            | uid1          | A      | A1        | 5    | 0     |
| def            | uid2          | A      | A1        | 10   | 1     |
| ghi            | uid3          | A      | A2        | 10   | 0     |
| jkl            | uid4          | B      | B1        | 5    | 0     |
| mno            | uid5          | B      | B1        | 10   | 1     |
| pqr            | uid6          | B      | B2        | 5    | 0     |
| stu            | uid7          | B      | B2        | 10   | 0     |
| wvx            | uid8          | C      | C1        | 10   | 1     |
| uza            | uid9          | C      | C1        | 10   | 0     |

Now, we show two different reports that could have been requested **by the advertiser**. Note that, as explained above, **the advertiser is only allowed one report, not the two of them**.

#### Example 1: Advertiser dimensions order 1 = [publisher_UID, domain, size, subdomain]

The report passed to the advertiser looks as follow:

| opportunity_ID | Publisher_UID | Domain | Size   | Subdomain | Label |
|----------------|---------------|--------|--------|-----------|-------|
| abc            | Hidden        | A      | Hidden | Hidden    | 0     |
| def            | Hidden        | A      | Hidden | Hidden    | 1     |
| ghi            | Hidden        | A      | Hidden | Hidden    | 0     |
| jkl            | Hidden        | B      | 5      | Hidden    | 0     |
| mno            | Hidden        | B      | 10     | Hidden    | 1     |
| pqr            | Hidden        | B      | 5      | Hidden    | 0     |
| stu            | Hidden        | B      | 10     | Hidden    | 0     |
| wvx            | Hidden        | C      | 10     | C1        | 1     |
| uza            | Hidden        | C      | 10     | C1        | 0     |

The advertiser has access to the opportunity ID as an unprotected variable. The publisher would not have access to it. The publisher would on its side have access to a display ID, again unprotected and only accessible to its side.

Publisher_UIDs are obviously all hidden as there are not k UIDs per UID. Note that this does not stop the algorithm and it returns "Hidden" as a feature value when for instance the value is missing.

All domains are available in clear because there are at least k displays for each domain.

Size: all sizes of domain A are hidden. Why? Because if indeed there are two displays on domain A with size 10, as there is only one remaining display (of size 5), this means that the "Hidden" report size is lower than k, and we must hide the size of displays done on domain A.

Subdomain: all are hidden except subdomains under the C domain, for the same reason as above.

The label is always available because the publisher has no access to the label, and therefore this does not need to be aggregated ("unprotected variable").

#### Example 2: Advertiser dimensions order 2 = [publisher_UID, domain, subdomain, size]

| opportunity_ID | publisher_UID | Domain | Subdomain | Size   | Label |
|----------------|---------------|--------|-----------|--------|-------|
| abc            | Hidden        | A      | Hidden    | Hidden | 0     |
| def            | Hidden        | A      | Hidden    | Hidden | 1     |
| ghi            | Hidden        | A      | Hidden    | Hidden | 0     |
| jkl            | Hidden        | B      | B1        | Hidden | 0     |
| mno            | Hidden        | B      | B1        | Hidden | 1     |
| pqr            | Hidden        | B      | B2        | Hidden | 0     |
| stu            | Hidden        | B      | B2        | Hidden | 0     |
| wvx            | Hidden        | C      | C1        | 10     | 1     |
| uza            | Hidden        | C      | C1        | 10     | 0     |

For similar reasons, a lot of categorical features are put to "Hidden", but here instead of the subdomain being hidden on B, it is the size. This shows the flexibility of this report, allowing the advertiser to choose which features are the most important to them.

### Continuous variables

Some important variables such as prices (in USD, or any currency) are continuous and thus don't fit well in a k-anonymous framework. Indeed, theoretically, the input space being infinite, there could many many cases of only one instance per value. None of these cases would pass the k-anonymity filter, no matter how low k is.

To get access to these variables, we propose to bucketize them. By lowering the input space, we add noise to the initial information but we make sure that multiple rows in the raw data can be considered of the same value and thus pass the k-anonymity filter.

In order to maximize the space reduction while minimizing information loss, the discretization of continuous space can be done intelligently and have a "denser bucketization" for often-used value ranges.

### Value proposition

This report allows for great flexibility, as it empowers actors to rank the features by order of importance according to them. If one advertiser is very interested in geolocation and another does not care, they can rank this dimension differently. The k-anonymity annihilates the incentive to create an individual domain per user and/or to hide PII within a feature since any granular grouping would be hidden in the report.

This also allows for and encourages the advertiser/publisher to find the right tradeoff between latency and precision (the longer the aggregation period is, the more granular the report would be).

Finally, it avoids hiding useful information to the advertiser/publisher that cannot be used to identify users anyway.

## Optional granular report on lost opportunities

The [ranked-privacy preserving granular report](#ranked-privacy-preserving-granular-report) only covers won auctions. To enable more transparency in the auction process, we propose to add a report on lost auctions for the advertisers. The report would have exactly the same features as the advertisers asked in the granular report, with only a few unprotected variables (eg interest group and AB-test id). The exact form of the report is open to suggestion, though we propose for now a report aggregated over the features asked. Reported variables would be the number of lost auctions, along with statistics regarding auction clearing price (ie mean and variance), with added differential privacy. The k-anonymity requirement should be kept, so as to avoid privacy attack from advertiser on lost auctions by asking for many features in the granular report (where most of them would be hidden, though not in the report on lost opportunities). As the number of lost auctions is very high compared to won ones, the k-anonymity threshold may have to be increased.


## Regarding conversion attribution

Conversion attribution to clicks is not covered by all the previous logs. This is due to the fact that the trusted third-party server does not have access to conversion events. Conversion attribution can be done in BINOCULARS in two ways:
- Link decoration: a click id is added as a decoration to the link, enabling the advertiser to link a subsequent conversion (if any) to the click. Note that only a click id is needed to attribute the conversion.
- Attribution done in browser: similar to [PCM](https://github.com/privacycg/private-click-measurement) or the [lift measurement](#lift-measurement) proposal of this document. The browser would keep a trace of all clicks and conversion, and do attribution. Reporting happens with some random delay after the sale to prevent timing attacks.

An industry-wide consensus has yet to form to decide which of the two solutions is an acceptable trade-off betweeen usability for all parties, user privacy and implicit consent.

# Appendix

## How does the algorithm used in the ranked privacy-preserving report to obfuscate values work?

The objective of this report is to pass as much information as possible to the advertiser and the publisher without allowing them to associate a user 1st party identity on the publisher side with a user 1st party identity on the advertiser side.

The advertiser ranks protected variables (like contextual data) by the order of importance (by its standards). They can also provide a set of transformations (truncation, bucketing, etc.). The reporting works as follows:

- The reporter (trusted third party server) counts the number of unique "pseudo-UIDs" (this is done in an anonymous way, using the ABTest_IDs) for each modality of the first variable. Please note that this count is done at the publisher domain level, and includes displays that were done on other interest groups and by other DSPs. **This means that one display may be enough for an advertiser to get contextual information, provided that others displays were done by other actors on similar contextual data**:

	- For all rows with a variable's modality (e.g. the URL domain of a given website) appearing more than a threshold k in the whole dataset, keep the actual value.
	- For all groups with a number of unique rows lower than k, return "Hidden" instead.

		- For this variable, should the number of "Hidden" rows in the group be lower than k, assign to "Hidden" the smallest group with size greater than k, so as to preserve privacy.
		- "Hidden" becomes the new "value" of the variable for the rows of this group, allowing the algorithm to continue if possible and have lower-ranked protected variables not hidden.
- The reporter groups by the n first variables he is asked to group by and verifies if the group still has a number of rows higher than k. Then it provides with the actual value or "Hidden" accordingly.
- Note that this process only takes protected variables into account. Other non-protected dimensions do not go through this procedure and always appear in clear.

The publisher report is produced using the same principle, based on the feature ranking provided by the publisher.
