# B.I.N.O.C.U.L.A.R.S.

This proposal is a work in progress and is meant to evolve in time. We welcome any feedback or comment: please raise new issues should you have any.

---

Relevant Internet advertising is an important component in striking a balance between providing a good end-user experience and allowing publishers and content creators to receive adequate compensation for the content they provide. Hence, the conjunction of user interests (as it is captured by interest groups in TURTLEDOVE) along with relevance with the current context  is a key driver of the open internet advertising ecosystem. Any weakness in leveraging both signals together would undoubtedly hurt both the publisher's revenue and the user experience, exposing them to irrelevant advertisements or worse, unsafe content.

Furthermore, advertisers want to ensure that their products are only appearing on websites that match their brand safety standards and, likewise, publishers want to ensure that all advertisements that appear on their site are suitable for their audience and their own brand image.

Reporting is a key aspect of any full-fledged proposal as the detailed reporting available on the web compared to other channels (billboards, TV ads, etc.) has been one of the main drivers of the advertisers' investment growth in the past decades. Reporting is used for a variety of use-cases ranging from close to real-time campaign management to verifying ex-post that the ad-quality rules have been respected.

Any proposal that does not allow joint use, and reporting of, publisher information along with the interest group is, by design, flawed and will lead to extremely limited adoption from the users, advertisers and publishers. B.I.N.O.C.U.L.A.R.S. - while taking many elements from Reporting in SPARROW - aims at laying out an independent reporting module that could be adopted to "watch" several birds in the aviary/proposals discussed at the W3C, including [FLEDGE](https://github.com/WICG/turtledove/blob/master/FLEDGE.md) as a replacement to the temporary event-level reporting. **While not specifically tied to SPARROW, these measurement reports fit in a privacy-safe ad delivery environment such as described in TURTLEDOVE and [fenced-frames](https://github.com/shivanigithub/fenced-frame)**. Its objective is to prevent any possible association between end-user PII - some being available on publisher side - and interest groups, whilst providing the best possible support for the variety of essential reporting use-cases, including AB testing and lift measurement.

# Executive Summary

Reporting relies on a combination of aggregated and [k-anonymous]((https://en.wikipedia.org/wiki/K-anonymity)) granular reports, computed and secured from a user privacy standpoint by the trusted third-party server. There are four different reports, with an optional fifth one:

1. A **near-real-time aggregated report**, with variable latency. This report should be "differentially private".
2. A **delayed served ads report** for publishers. It informs about the creatives and origins of the ads such that the publisher can check if the served ads comply with its ads policy (i.e. publisher ad quality). To meet privacy constraints, a sampling mechanism could be added.
3. A **delayed ranked privacy-preserving granular report based on k-anonymity**:

	- This report is granular, meaning one row per display.
	- A granular report with intentionally bucketed continuous variables and k-anonymity on variables shared by advertiser and publisher (information about the interest group is never available to the publisher, even with appropriate k-anonymity).
	- A different version of this report is available for the advertiser and the publisher.
4. An **aggregated conversion report**, including conversions by users in the control population of the lift AB test (who, by definition didn't see ads) who had opportunities to receive one.
5. An optional **aggregated report on lost opportunities**, to enable actors to learn about lost auctions.

AB testing and lift measurement use a cross-channel framework. This allows for advanced AB testing, essential to all actors across the web, and fair comparisons between marketing vendors/channels, etc. This would bring much-needed clarity and transparency in measuring performance and would be a desirable improvement compared to the current situation.


# Links to the detailed propositions

* [**Reporting in BINOCULARS**](./reports.md)
* [**AB-testing in BINOCULARS**](./AB-testing.md)


