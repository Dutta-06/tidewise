# Demo video script (target 2:45, hard limit 3:00)

The submission asks for a published or unlisted video of **no more than 3 minutes**
covering: about the project, tech stack and architecture, how AWS is used, and
(optionally) learning and growth. The sections below follow that order. The target is
2:45 so a slow take still finishes inside the limit.

**Every number in brackets must come from a real run** (the deployed stack, the replay
rerun on real history and the real trace, and `forecasting/models/metrics.json`). Do not
record with the synthetic numbers. Only claim what is deployed: check CloudFront and the
number of worker regions in `HUMAN-TODO.md` before saying them.

Record the screen and the AWS console separately; cut on a timeline. Upload to YouTube as
**Unlisted** (or Public) and open the link in a private window to check it plays.

## 0:00-0:40 About the project
**Visual:** the water-per-kWh against wet-bulb curve (`docs/cost-model.md`), then the live
dashboard (`https://main.d38pxhs7tqujak.amplifyapp.com/`): Submit a 4 GPU-hour job with a
deadline on Friday, then the Job page with the explanation sentence and the two headline
numbers.

> "Data centres cool themselves by evaporating water, and the hotter and more humid it is
> outside, the more they use. One kWh of AI compute can cost [2.2] litres of water on site
> in a Delhi afternoon and close to [zero] on a Nordic night. Most AI work isn't urgent:
> training, fine-tuning and evaluation sweeps have deadlines in hours or days. Tidewise
> uses that slack. It picks the AWS region and the hour that finish before the deadline
> with the least water and carbon, explains why, and issues a receipt: [X]% less water and
> [Y]% less CO2 than running now in Mumbai."

## 0:40-1:15 Tech stack and architecture
**Visual:** a single tech-stack slide (logos or labels, grouped), held for about 8 seconds,
then the diagram from `docs/architecture.md`.

Slide contents:
- **Backend:** Python 3.12 on AWS Lambda, defined with AWS SAM
- **Data:** Open-Meteo (weather), Electricity Maps (grid carbon)
- **Forecasting:** gradient-boosted trees (scikit-learn) judged against persistence and the
  provider, exported to JSON so Lambda needs no ML runtime
- **Scheduler:** deadline-constrained greedy search, OR-Tools CP-SAT for splittable jobs
- **Frontend:** React, Vite, Leaflet
- **Assistant:** Strands agent calling Claude over the public API
- **Cloud:** AWS (next section)

> "The backend is Python on Lambda. Every hour a pipeline pulls weather and grid carbon
> for [8] regions and forecasts wet-bulb temperature and carbon 48 hours ahead. The model
> is only used where it beats a naive baseline. A deadline-constrained scheduler prices
> every region-hour in litres and kilograms and picks the best slot. The dashboard is React
> with Vite and Leaflet, and an optional Strands-based assistant can submit jobs in plain
> English."

## 1:15-2:15 How AWS is used
**Visual:** the real AWS console, a few seconds each, in this order. The Step Functions
graph gets the most time.

1. **EventBridge + Step Functions** (`pravaah-forecast`): "An EventBridge rule starts a
   Step Functions pipeline every hour that fetches, predicts and stores, one branch per
   region, so one failing region doesn't stop the rest."
2. **DynamoDB + S3:** "Forecasts, jobs and receipts live in DynamoDB. Default price
   surfaces are published to S3 so the dashboard's standard views never touch Lambda."
3. **API Gateway + Lambda:** "API Gateway and about [20] Lambda functions serve the
   dashboard, with Cognito protecting the team policy routes."
4. **Step Functions** (`pravaah-run`, graph with the Wait state): "Each job gets its own
   state machine. It waits until the chosen hour, then invokes a worker Lambda in the
   chosen region and writes the receipt."
5. **Receipt** showing `ran_in: [eu-north-1]` and `launched_via: cross-region-lambda`.
6. **SNS + CloudWatch dashboard:** "SNS emails the result. CloudWatch alarms watch the
   pipeline, and a dashboard tracks litres saved."
7. **Amplify:** "Amplify Hosting serves the dashboard, and the whole backend is one SAM
   template."

> "Honest receipts: Lambda has no GPUs, so the declared GPU job's energy is estimated and
> the small proxy workload that actually ran is measured. The receipt labels which is
> which."

## 2:15-2:35 Does it add up?
**Visual:** Savings page, trace replay bars, then the when-versus-where table.

> "Replaying [500] jobs from [Alibaba's public GPU cluster trace]: [X]% less water, [Y]%
> less CO2, [100]% of deadlines met, median delay [N] hours. Most of the saving comes from
> where a job runs, not when."

## 2:35-2:50 Learning and growth (optional, keep it short)
**Visual:** Forecast quality page, then the Policies page weights.

> "I learned that water and carbon don't always agree: a nuclear-powered grid is clean but
> thirsty, so users set the weights. I also learned to be honest about what we can't
> measure. Cooling water is modelled from published curves and calibrated to AWS's own
> disclosures, so absolute litres are uncertain, but the ranking of slots holds up."

## 2:50-3:00 Close (cut this card first if you run long)
> "We used AI to decide when AI should run."

**End card:** repo URL, live URL, team names.
