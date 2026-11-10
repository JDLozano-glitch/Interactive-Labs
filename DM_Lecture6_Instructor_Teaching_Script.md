# DATA 502 — Lecture 6 Instructor Teaching Script
## Association rules and co-occurrence

**Audience:** Graduate Diploma in Data Science students  
**Suggested duration:** 90 minutes, including discussion and live coding  
**Notebook:** `DM_Lecture6_Lecture_Association_Rules.ipynb`  
**Teaching mode:** Interactive lecture with short pair discussions, prediction pauses, and live coding.

> **Instructor note:** This is a delivery guide, not a word-for-word requirement. The quoted prompts can be read aloud. Pause for student thinking before revealing answers or running demonstrations. Keep emphasizing that association rules describe co-occurrence in the chosen transaction representation; they do not establish cause, time order, or practical value by themselves.

## Preparation

1. Open the notebook in Google Colab or Jupyter and run the setup cell before class.
2. Confirm that the toy basket, Iris transaction matrix, and `pair_rules` cells run from a fresh kernel in order.
3. Display the learning outcomes and have a whiteboard or shared space ready for three formulas:
   - **Support**: count(A and B) / total transactions
   - **Confidence(A → B)**: count(A and B) / count(A)
   - **Lift(A → B)**: confidence(A → B) / support(B)
4. Remind students that the toy baskets are invented; Iris is a real historical dataset, but its bins and rules here are only teaching representations.
5. Keep an eye on time. If needed, shorten pair-share reporting rather than skipping the distinction between confidence and lift or the caveats on interpretation.

## At-a-glance timing

| Time | Notebook section | Teaching focus |
|---|---|---|
| 0–5 min | Welcome and outcomes | Connect association rules to descriptive data mining |
| 5–15 min | A01–A05 | Applications, diagnostic, transaction units, itemsets and rules |
| 15–25 min | A06–A08 | Toy baskets, support and confidence |
| 25–35 min | A09–A11 | Directionality, lift and formula cross-check |
| 35–45 min | A12–A14 | Thresholds, candidate rules, small samples and multiple searches |
| 45–65 min | A15–A18 | Iris feature binning, matrix, pair-rule code and interpretation |
| 65–75 min | A19 | Ethics and practical usefulness |
| 75–82 min | A20 | Brief the Wine lab |
| 82–90 min | A21 | Exit check, recap and transition |

## Delivery script

### 0–5 min — Welcome and framing

**Say:** “Today we move from grouping observations to finding items or attributes that appear together. The central question is not simply ‘What is the rule?’ It is also ‘What counts as a transaction?’, ‘Compared with what baseline?’, and ‘How much evidence supports it?’”

**Say:** “An association rule is descriptive. The arrow is a convenient way to express a conditional proportion; it does not mean that the antecedent happened first or caused the consequent.”

Briefly read the outcomes in the notebook. Ask students to keep a note of any term they are unsure about. Explain that they will see a small example first, calculate measures by hand, then inspect an actual pandas implementation.

### 5–15 min — A01–A05: Conceptual foundation

#### A01 — From clusters to associations (about 3 min)

**Say:** “Clustering asks which observations are similar. Association-rule mining asks which items co-occur across a collection of transactions. Both can help us describe data, but neither alone tells us why a pattern exists.”

**Ask:** “Think of one domain where co-occurrence could be useful. What is the transaction in that setting?”

Allow one minute of pair discussion, then invite two examples. For each, ask the speaker to identify the transaction unit. If students say “customers who buy X also buy Y,” ask whether the transaction is a receipt, a customer’s whole history, or a time window for one customer.

**Transition:** “That choice changes the counts, so we need to define it before calculating anything.”

#### A02 — Learning outcomes (about 1 min)

Ask students to privately rate their confidence from 1 to 5. Tell them they will revisit the rating at the end. Emphasize that the goal is not only to compute but to explain what each measure means and what it cannot establish.

#### A03 — Opening diagnostic (about 3 min)

Read the three diagnostic statements: high confidence always proves usefulness; a rule implies causation; and reversing a rule gives the same rule.

**Say:** “Choose a position for each claim and be ready to give a reason. Do not worry about being right yet; we will return to these.”

Take a quick show of hands or poll. Avoid resolving every claim immediately. Confirm that students should be suspicious of “always,” and explain that the lecture will provide the denominators and baseline needed to assess the claims.

#### A04 — Transactions and items (about 2 min)

**Say:** “A transaction is the unit we count once. An item is a binary event or attribute recorded as present in that unit. In a basic basket matrix, an item is either present or absent; buying three units of tea does not make tea count three times.”

Ask: “How would the interpretation change between one receipt and one customer’s entire year?”

Expected response: receipt-level patterns mean items bought on the same visit; customer-level patterns mean items bought at any point in the period. Longer aggregation can inflate apparent co-occurrence and discards timing and visit context.

#### A05 — Itemsets and rules (about 2 min)

Point to `{tea} → {biscuit}`. Ask students to name antecedent and consequent and translate the rule.

**Expected wording:** “Among transactions containing tea, some proportion also contain biscuits.”

**Say:** “Do not translate this into ‘tea causes biscuit purchases’ or ‘tea is bought first.’ The basket has no causal or temporal design.”

### 15–25 min — A06–A08: Toy baskets, support and confidence

#### A06 — Predict the basket counts (about 3 min)

Read the five baskets aloud or point to the notebook list. Give students 30 seconds to write counts before running code.

**Ask:** “How many transactions contain tea? Biscuits? Both?”

**Expected answer:** Tea = 3; biscuits = 3; both = 2; total = 5.

Have a student explain what the rows and columns in the matrix will represent before executing the cell.

#### Live code: toy transaction matrix (about 2 min)

Run the cell. Point out that `True` means the item is present and `False` means absent; the display converts these to 1 and 0 for readability.

**Say:** “The code is a mechanical representation of our stated transaction definition. It does not decide what a meaningful basket is; we did that first.”

Check that the displayed counts match the predictions. If not, trace one row together.

#### A07 — Support (about 2 min)

Write `2/5 = 0.40 = 40%` on the board.

**Say:** “Support answers: what fraction of all transactions contain both items? Its denominator is the total number of transactions. In a store with 1,000 comparable baskets, 40% would correspond to 400 baskets, assuming this small example’s rate generalized—which it may not.”

Ask what support does not tell a manager. Expected points: not conditional behavior among tea buyers, not causation, not profitability, and not whether the sample represents future customers.

#### A08 — Confidence (about 3 min)

Write `confidence(tea → biscuit) = 2/3 ≈ 66.7%`.

**Say:** “Now the question changes: among the baskets that contain tea, what fraction also contain biscuits? The denominator is tea baskets, not all baskets.”

Ask a student to explain the result in ordinary language. Then ask whether 100% confidence based on one tea basket would be convincing. Guide them toward the need for counts, a baseline, and validation.

### 25–35 min — A09–A11: Direction, lift and cross-check

#### A09 — Direction matters (about 3 min)

Use the notebook’s hypothetical counts: A occurs in 4 baskets, B in 2, both in 2.

Ask students to calculate both directions independently.

**Expected answers:** confidence(A → B) = 2/4 = 0.50; confidence(B → A) = 2/2 = 1.00.

**Say:** “The joint count is the same in both directions. The denominator changes because the antecedent changes. So reversing a rule can change confidence substantially.”

#### A10 — Lift and baseline (about 4 min)

Write the lift definition and then calculate for the toy data:

- confidence(tea → biscuit) = 2/3
- support(biscuit) = 3/5
- lift = (2/3)/(3/5) = 10/9 ≈ 1.11

**Say:** “Lift compares the conditional rate with the consequent’s overall rate. Here biscuit appears in 60% of all baskets, and in about 66.7% of tea baskets. The conditional rate is about 1.11 times the baseline.”

Explain the reference points: lift 1 corresponds to empirical independence; above 1 is positive association; below 1 is negative association. Avoid describing lift as the probability of buying the consequent.

**Ask:** “Could confidence be high while lift is near 1?”

**Expected answer:** Yes. If the consequent is already very common, its rate among antecedent transactions can be high without being much higher than its baseline.

#### Live code: toy measures (about 2 min)

Before running the cell, ask students to identify the denominator for each printed measure. Run it and point out `n`, `a.sum()`, and `b.mean()`.

**Say:** “Notice that the code follows the definitions: joint count over all transactions for support; joint count over antecedent count for confidence; and confidence divided by the consequent baseline for lift.”

#### A11 — Cross-check (about 1 min)

Show the alternative formula `support(A and B) / (support(A) × support(B))`. Explain briefly that it is symmetric for the pair, which is why reversing a pair leaves lift unchanged. Note that zero denominators make some measures undefined and must be handled explicitly.

### 35–45 min — A12–A14: Thresholds and reliability

#### A12 — Frequent itemsets (about 3 min)

**Say:** “Frequent means that an itemset meets a chosen minimum-support threshold. It does not mean the pattern is important, useful, or true in every population.”

Ask whether a pair with 20% support can contain an item with only 10% support. Expected answer: no; each item in the pair must occur in at least all transactions containing the pair, so its support is at least 20%.

Discuss why thresholds are choices: computation, practical prevalence, rare events, and the cost of missing a pattern all matter.

#### A13 — Candidate rules (about 3 min)

Explain that `{A, B}` generates two directed rules, while confidence thresholds are just one filter. Ask for examples of a high-confidence but unhelpful rule.

Expected possibilities: a very common consequent; a tiny joint count; an action that is too expensive; a rule already known; a rule that does not change a decision; or a relationship caused by how the data were collected.

#### A14 — Small samples and many searches (about 4 min)

**Say:** “A striking ratio can be built from very few observations. Also, if we search thousands of candidate rules, some will look unusual by chance.”

Ask: “Would you act on lift 5 from two joint transactions?”

Guide toward: not immediately; inspect counts and uncertainty, check data quality and selection, define a follow-up test, and validate on a separate or later dataset. Mention that rule selection itself should be reported so others can understand how the candidate was found.

### 45–65 min — A15–A18: Iris live demonstration

#### A15 — From numerical features to transactions (about 3 min)

**Say:** “Association rules are often introduced with shopping baskets, but the same representation can be applied to discretized measurements. Here, each flower is one transaction, and each measurement becomes a high or low item.”

Emphasize that the median threshold is a teaching choice. It discards information and the rule meanings depend on the bins. Species is intentionally excluded from the item baskets.

Ask what could change if the cutoff changed. Expected answer: item presence, item frequencies, joint counts, and therefore every rule measure could change.

#### Live code: load Iris and inspect medians (about 2 min)

Run the cell. Point out that `iris.data` is used and species labels are not included. Have students read one median and its units. Clarify that the displayed label says cm for the four Iris features.

#### A16 — One-hot transaction matrix (about 3 min)

Ask students to predict dimensions: 150 rows and 8 columns (two items for each of four measurements).

**Say:** “For each measurement, a flower is assigned to exactly one category. Equality to the median goes into HIGH because the code uses `>=`; LOW uses `<`.”

Run the cell and inspect the first row. Ask why HIGH and LOW for the same measurement cannot both be true for a row.

#### A17 — Explain and run the rule code (about 7 min)

Before execution, point out that 8 items yield `8 choose 2 = 28` unordered pairs and two directions per pair, or up to 56 directed candidates before filtering.

Walk through the function in this order:
1. `combinations` chooses each unordered item pair once.
2. `ca` and `cb` count each item’s transactions.
3. `joint` counts transactions containing both.
4. `min_count` excludes pairs with too few joint observations.
5. The inner loop creates both directions.
6. Support, confidence, baseline and lift use the denominators already discussed.
7. The final filter keeps lift above 1 and sorts for display.

Run the cell. Explain that it is a simple pair-rule demonstration, not a general optimized algorithm for all possible itemsets. Ask students to find a value in the output and state its denominator before interpreting it.

#### A18 — Interpret the actual output (about 5 min)

Run the interpretation cell. If the result table is empty, do not treat that automatically as an error; discuss the joint-count and lift settings. Otherwise, read the first rule using the complete template:

“Among 150 flowers, **[joint count]** had both **[antecedent]** and **[consequent]**. Of the **[antecedent count]** flowers with **[antecedent]**, **[confidence × 100]%** also had **[consequent]**. The consequent appeared in **[baseline × 100]%** overall, giving a lift of **[lift]**.”

Then add: “This is an exploratory descriptive pattern in this dataset and under these median bins; it does not show that one measurement causes another.”

Ask a pair to identify one missing caveat or piece of context. Reinforce the need to report the population, transaction unit, bin thresholds, counts and filtering choices.

### 65–75 min — A19: Ethics and practical usefulness

**Say:** “A technically correct rule can still be inappropriate to use. Co-occurrence may reflect who was measured, what was recorded, a proxy for a sensitive attribute, or an institutional process.”

Ask groups to consider the notebook prompt about a sensitive personal attribute. Give them two minutes, then ask:
- Who could benefit from acting on the rule?
- Who could be harmed or excluded?
- Could a seemingly harmless variable act as a proxy?
- What data minimization, consent, review, or validation would be needed?

Do not treat removal of a sensitive column as a complete solution. Emphasize intended use, governance, and impact on affected groups.

### 75–82 min — A20: Wine lab briefing

Explain the next lab’s task: use Wine samples as transactions, select four measurement features, create median-based binary items, exclude class from the baskets, calculate rules, compare directions, and write a cautious interpretation.

Ask students to list what must be recalculated. Expected answer: feature cutoffs, item counts, joint counts, support, confidence, baseline, lift, and qualifying rules. They should also justify transaction unit, binning, minimum counts/thresholds, and validation approach.

Clarify that the Iris rule values cannot be carried over to Wine; only the definitions and workflow transfer.

### 82–90 min — A21: Exit check and closing

Give students two minutes to solve independently:

- Total baskets = 20
- A count = 8
- B count = 10
- Joint count = 4

**Expected answers:**
- Support(A → B) = 4/20 = 0.20
- Confidence(A → B) = 4/8 = 0.50
- Support(B) = 10/20 = 0.50
- Lift = 0.50/0.50 = 1.00

**Interpretation:** “Twenty percent of all baskets contain both A and B. Half of baskets containing A also contain B, equal to B’s 50% overall baseline; the observed pair has lift 1.”

Ask: “Does this establish that A and B are unrelated in every population or that they have no practical connection?”

Expected answer: no. It only indicates no observed deviation from independence under this dataset and transaction definition; uncertainty, sample representativeness, subgroup structure and context remain relevant.

Close by revisiting the diagnostic. Ask students to correct each opening claim. Invite them to update their confidence ratings and identify one idea they will apply in the Wine lab.

## Answer key and common misconceptions

| Prompt or idea | Expected response / clarification |
|---|---|
| Toy basket counts | 5 total; tea 3; biscuit 3; both 2. |
| Toy support | 2/5 = 0.40. Denominator is all transactions. |
| Toy confidence, tea → biscuit | 2/3 ≈ 0.667. Denominator is tea transactions. |
| Toy confidence, biscuit → tea | Also 2/3 here only because both marginal counts equal 3. |
| Toy lift | (2/3)/(3/5) = 10/9 ≈ 1.11. Consequent baseline is 0.60. |
| Hypothetical direction question | A → B = 2/4 = 0.50; B → A = 2/2 = 1.00. |
| Pair support vs item support | Each item’s support must be at least the support of a pair containing it. |
| Exit check | Support 0.20; confidence 0.50; B baseline 0.50; lift 1.00. |
| “High confidence means useful” | Not necessarily; compare the consequent baseline, counts, costs, context and validation. |
| “A → B means A causes B” | No. It is a conditional co-occurrence summary, not causal evidence. |
| “A → B equals B → A” | The joint count and pair lift are the same; confidence is generally directional. |
| “Lift above 1 proves a dependable pattern” | No. Small samples, multiple comparisons, selection and shifting data can mislead. |

## Contingency notes

- **If Colab is slow or unavailable:** Work through the toy counts on the board and use screenshots or a prepared output for Iris. Preserve the prediction and interpretation questions.
- **If the Iris rules differ from a prepared example:** Results may vary if code, versions, data, or filters differ. Interpret the output generated in class and inspect the matrix and thresholds rather than relying on memorized rule values.
- **If no rules pass the filters:** This is an opportunity to explain the role of thresholds. Lowering a threshold is not automatically better; ask what is being traded off.
- **If time is short:** Keep the toy calculation, lift baseline explanation, one code walkthrough, actual-rule interpretation, ethics check and exit calculation. Reduce the number of student report-outs.
- **If students finish early:** Ask them to change `min_count`, compare the rule table, and explain how changing the filter affects the set of displayed candidates without changing the underlying transaction matrix.
