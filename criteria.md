# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
I chose 4 out of 5 because the outfit and fit card tools depend on the model. An occasional API error or empty response could prevent a matching query from finishing, but the agent should still complete the full process reliably in most runs.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
I chose 5 out of 5 because this path uses a simple Python check, not a model response. When the search returns an empty list, the agent should always stop and tell the user what they can change.

---

## 3. Something about state

When a search finds a match, the item ID saved in session["selected_item"] should match the item ID passed to suggest_outfit in 5 out of 5 runs.



**Why this target:**
I chose 5 out of 5 because the code should pass the same saved item to the outfit tool every time. The model may change its wording, but it should not affect which item the tool receives.


---

## 4. Something about the fit card

For five runs using selected listings, at least 4 out of 5 fit cards contain no incorrect or invented brand, color, or price details when compared with the selected listing. If the listing's brand is null, the caption must not name a brand.



**Why this target:**
I chose 4 out of 5 because the model can use different wording and may occasionally add an unsupported detail. Accurate information matters because the caption should describe the actual listing, including leaving out the brand when it is unknown.


---

## 5. Your choice

Across five searches with a specified max_price and at least one known matching listing within budget, search_listings returns a non-empty list, and every returned item's price is less than or equal to max_price — in 5 out of 5 searches.



**Why this target:**
I chose 5 out of 5 because the price filter is a Python comparison and should work consistently. A user should not receive items above their budget, and the search should still return matching items that are affordable.


---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
