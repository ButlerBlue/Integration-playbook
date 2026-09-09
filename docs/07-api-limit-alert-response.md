# Responding to an Org's API Limit Alert

*What to do when a client's CRM org's API request volume is approaching or has hit its daily limit.*

## Introduction

Every Salesforce org has a maximum number of API requests per 24-hour window. Salesforce typically offers a brief grace window once that limit is hit before rejecting additional requests outright. Once requests start getting rejected, every integrated service fails, for example, an integrated web form tool will start throwing errors for end users trying to submit it. This blocks real business processes and generates a spike in support tickets, so the goal is to catch and act on this before an org actually hits 100% utilization.

A solid implementation prevents this three ways:
1. Configure a usage alert (e.g., at 80% utilization) as a standard part of onboarding, so the account owner is notified with room to act.
2. On alert, identify which API consumer is driving the spike and ask them to resolve it on their end quickly.
3. If they can't act fast enough, disable that app or user's connection before the org hits 100%, rather than letting the whole org go down for everyone.

The rest of this guide covers how to execute that strategy.

## Instructions

The account owner is responsible for client and partner/integrator communication throughout this process.

1. **Confirm the alert is real.** Check your platform's system overview (in Salesforce: Setup > System Overview) to confirm API requests are actually maxing out. Share a screenshot with your team so everyone has visibility that it's being actively worked.

2. **Confirm the trend with your monitoring tooling** (e.g., Datadog or an equivalent org health dashboard). Look at the rate of increase, typically shown over the last several days, and share that with the team as well so the urgency is clear.

3. **Identify the consumer.** Pull an API usage report by user for the last several days (note: exclude any database sync connector like Heroku Connect from this analysis, since that traffic doesn't count against the API limit). Keep in mind these counts typically reset at midnight UTC, so timing matters when reading the numbers. Share a screenshot highlighting whoever is most consumptive.

   *Tip: Salesforce Classic supports direct URL access to standard reports using the object's key prefix. The API usage-by-user report can often be reached directly this way if you know the report's internal ID, worth bookmarking once you find it for your org.*

4. **Escalate internally.** If usage is maxed or maxing out, loop in your implementation lead and anyone else who needs visibility, so they can weigh in with next steps.

5. **If the consumer turns out to be a partner or third-party integrator:**
   - Notify your partner services team so they're aware.
   - Reach out to the integrator directly with a clear, time-bound ask to resolve the issue on their end.
   - If you don't get a quick response committing to a fix, send a warning that access will be deactivated.
   - If there's still no response, deactivate their access rather than let the whole org's API usage collapse.

6. **Keep monitoring** the same resources from the earlier steps to confirm usage is coming back down, and keep the team updated as it resolves.
