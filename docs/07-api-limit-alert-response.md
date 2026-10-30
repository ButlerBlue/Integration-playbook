# Responding to an Org's API Limit Alert

*What to do when a client's CRM org's API request volume is approaching or has hit its daily limit, and how to tell that apart from a storage limit before acting.*

## Introduction

Every Salesforce org has a maximum number of API requests per 24-hour window. Salesforce typically offers a brief grace window once that limit is hit before rejecting additional requests outright. Once requests start getting rejected, every integrated service fails, for example, an integrated web form tool will start throwing errors for end users trying to submit it. This blocks real business processes and generates a spike in support tickets, so the goal is to catch and act on this before an org actually hits 100% utilization.

A solid implementation prevents this three ways:
1. Configure a usage alert (e.g., at 80% utilization) as a standard part of onboarding, so the account owner is notified with room to act.
2. On alert, identify which API consumer is driving the spike and ask them to resolve it on their end quickly.
3. If they can't act fast enough, disable that app or user's connection before the org hits 100%, rather than letting the whole org go down for everyone.

The rest of this guide covers how to execute that strategy.

## First, Confirm It Is the API Limit

Vendors and colleagues often describe any "limit exceeded" error as hitting the API limit. Salesforce tracks several limits independently, and the fix for one does nothing for another. Read the status code before acting.

| Status code | What ran out | What fixes it | What doesn't |
|---|---|---|---|
| `REQUEST_LIMIT_EXCEEDED` | The org's rolling 24-hour API request allowance | Reducing call volume, batching, or disabling the consumer, as in the steps below | Deleting data |
| `STORAGE_LIMIT_EXCEEDED` | The org's data storage (or file storage) allocation | Freeing storage, permanently deleting test data, or loading less | Waiting for the API counter to reset |

This matters most during vendor testing. A vendor validating a large export or load in a sandbox can fill the sandbox's storage allocation, which is much smaller than production's, long before they come near the request limit. If the response says storage, the request limit hasn't been tested at all, and the vendor's production sizing question is still open.

To confirm a storage problem:

1. **Open Setup > Storage Usage** in the affected org. It shows current data and file storage against the allocation and breaks usage down by object, so you can see whether one object, usually the one the vendor was loading, accounts for most of it.
2. **Check the Recycle Bin, but don't expect emptying it to free storage.** Salesforce excludes Recycle Bin contents from Data Storage usage, so purging it won't move the Storage Usage number. If a vendor says they deleted their test batch and Storage Usage still shows it, look for records that are still live rather than deleted: a filtered delete that missed related child records or attachments, a different object than the one they think they cleared, or File Storage (Files, Documents, attachments), which is tracked separately from Data Storage.
3. **If the org is over its allocation, not just near it,** have the vendor permanently delete enough of the test batch to get back under before running anything else. New writes will keep failing until then.
4. **Right-size the test data.** A few thousand representative rows is usually enough to validate batching and performance in a sandbox. Reserve full-volume testing for an environment sized for it, such as a full-copy sandbox.

To confirm a request limit problem, use the steps under Instructions below, or query the REST API's limits resource (`/services/data/vXX.X/limits`), which reports current usage against the maximum for each governed limit, including `DailyApiRequests`.

### Helping a Vendor Test Limit Handling Safely

A vendor who wants to prove their integration handles both limits doesn't need to hit either one for real.

1. **Mock both responses in their own test suite.** Return a `REQUEST_LIMIT_EXCEEDED` and a `STORAGE_LIMIT_EXCEEDED` error from a stub and confirm the integration backs off, logs clearly, and doesn't retry in a tight loop. No platform interaction is needed.
2. **If they want to see the request limit for real, drive it with cheap calls,** such as a repeated simple query, never by creating records. The request counter counts calls, not what the calls do.
3. **Have them poll the limits resource during a normal run** to measure real call volume against the allowance. That answers the production sizing question better than forcing a failure.
4. **Ask them to handle the two codes differently.** A request limit error means wait and slow down. A storage error means stop and tell a person, because no amount of retrying will fix it.

## Instructions

The account owner is responsible for client and partner/integrator communication throughout this process.

1. **Confirm the alert is real.** Check your platform's system overview (in Salesforce: Setup > System Overview) to confirm API requests are actually maxing out. Share a screenshot with your team so everyone has visibility that it's being actively worked.

2. **Confirm the trend with your monitoring tooling** (e.g., Datadog or an equivalent org health dashboard). Look at the rate of increase, typically shown over the last several days, and share that with the team as well so the urgency is clear.

3. **Identify the consumer.** Pull an API usage report by user for the last several days (note: exclude any bi-directional database sync connector from this analysis, since that traffic doesn't count against the API limit). Keep in mind these counts typically reset at midnight UTC, so timing matters when reading the numbers. Share a screenshot highlighting whoever is most consumptive.

   *Tip: Salesforce Classic supports direct URL access to standard reports using the object's key prefix. The API usage-by-user report can often be reached directly this way if you know the report's internal ID, worth bookmarking once you find it for your org.*

4. **Escalate internally.** If usage is maxed or maxing out, loop in your implementation lead and anyone else who needs visibility, so they can weigh in with next steps.

5. **If the consumer turns out to be a partner or third-party integrator:**
   - Notify your partner services team so they're aware.
   - Reach out to the integrator directly with a clear, time-bound ask to resolve the issue on their end.
   - If you don't get a quick response committing to a fix, send a warning that access will be deactivated.
   - If there's still no response, deactivate their access rather than let the whole org's API usage collapse.

6. **Keep monitoring** the same resources from the earlier steps to confirm usage is coming back down, and keep the team updated as it resolves.
