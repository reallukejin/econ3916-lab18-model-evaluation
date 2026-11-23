# econ3916-lab18-model-evaluation

# Model Evaluation: The Curve, and What It Cannot Tell You

## Objective
I wanted to see what ROC and precision-recall curves can and cannot tell me about a classifier for a rare outcome, and how a business cost turns a curve into an actual decision.

## Methodology
- I simulated 8,000 cases of a rare outcome from a known logistic model with five features (one of them pure noise), and kept 30% of the rows for testing, stratified so both sets had the same share of positives.
- I fitted a scaled logistic regression on the training rows and drew its ROC curve and precision-recall curve on the test rows, with ROC AUC and average precision as the one-number summaries.
- I fitted a second model, a gradient-boosted classifier (120 trees, depth 2), and compared the two on ROC AUC, average precision and precision at four recall levels.
- I ran a paired bootstrap over 500 resamples of the test set, scoring both models on each resample, to put a 95% interval around the gap between them.
- I built an expected-cost curve where a miss costs ten times a false alarm, chose the cheapest threshold on the training rows using cross-validated probabilities, and priced that threshold once on the test rows.
- I swept the cost ratio from 2:1 to 50:1 for both models, choosing each threshold the same way, to see whether the cheaper model changes.
- I wrote a small function, roc_point(), that flags every row at or above a threshold and returns that single point on the ROC curve (false positive rate, true positive rate).

## Key Findings
- 11.8% of the test rows were positive (283 of 2,400).
- The logistic regression had a ROC AUC of 0.847 but an average precision of only 0.465. At low prevalence, the ROC curve's false positive rate divides by a large pool of negatives, so it looks good even when false alarms swamp the true positives. Average precision is still about four times the 0.118 a random classifier would get.
- The boosted model had a ROC AUC of 0.8431. The AUC difference on the full test set was +0.0038, and the paired bootstrap gave a 95% interval of [−0.0020, +0.0094]. Since the interval includes zero, I can't tell the two models apart on this test set.
- The cheapest threshold chosen on the training rows was 0.0905. On the test rows it cost 0.4692 per case, against 0.9437 at the default 0.5, so the default threshold roughly doubles the cost. The cost curve is fairly flat near its minimum, so the business only needs to state its cost ratio roughly, not exactly.
- Across the cost ratios, the logistic model was cheaper at every ratio except 5:1, but the gaps were small and within the noise the bootstrap showed, so I would not act on them.
- At my chosen threshold, roc_point() gave a false positive rate of 0.296 and a true positive rate of 0.823.
- The main lesson: the curves rank models, but they can't pick a threshold or a winner on their own. That takes a cost ratio, which comes from the business, not from the data.
