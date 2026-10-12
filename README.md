# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools and the planning loop are implemented. The ask command
> searches local listings, suggests an outfit, and returns a fit card.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

FitFindr helps users find secondhand clothing from a local listings file by describing what they want, their size, and their budget. It ranks matching listings and selects the first result, then uses a model to suggest outfits with pieces from the user's wardrobe. It also creates a short caption about the selected item and outfit. If no listings match, it stops and suggests changing the search description, size, or budget.


---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Filters local listings by size and budget, then ranks them by the number of matching description keywords found in the title, description, and style_tags, ignoring capitalization.
- **Inputs:** `description` (str), `size` (str or None; None means any size), `max_price` (float or None; None means no price limit).
- **Returns:** A list of listing dictionaries containing id, title, description, category, style_tags, size, condition, price, colors, brand, and platform. Results match at least one description keyword, satisfy all supplied filters, and are ordered by keyword score with original data order breaking ties. Returns at most `config.SEARCH_RESULT_LIMIT` items. Size matching accepts exact sizes and slash-separated alternatives: M matches S/M, but L does not match XL. Prices must be less than or equal to max_price.
- **When it has nothing:** Returns an empty list (`[]`) when no listings match.

### `suggest_outfit`

- **What it does:** Calls the model to suggest an outfit pairing the selected listing with items from the user's wardrobe.
- **Inputs:** `new_item` (dict containing a listing), `wardrobe` (dict with an `items` list of wardrobe item dictionaries).
- **Returns:** A non-empty string describing an outfit that includes the new item, names wardrobe pieces to pair with it, and explains why they work together.
- **When it has nothing:** If `wardrobe["items"]` is empty, returns general styling advice for the new item without claiming the user owns any suggested pieces.

### `create_fit_card`

- **What it does:** Calls the model to write a short, post-ready caption about the new item and suggested outfit.
- **Inputs:** `outfit` (str containing outfit advice), `new_item` (dict containing a listing).
- **Returns:** A non-empty string containing a caption that describes the new item and outfit without inventing listing details.
- **When it has nothing:** If the outfit is blank or the new item is empty, returns "Cannot create a fit card without an outfit and a selected item." without calling the model.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If search_listings returns an empty list, save a message in the session suggesting the user change their description, size, or budget, then stop without calling suggest_outfit or create_fit_card. Otherwise, save the first matching listing as selected_item in the session, pass it from the session to suggest_outfit, save the outfit in the session, and use the saved outfit and selected_item to call create_fit_card. Save the resulting caption in the session.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** `agent.py::parse_query` uses regular expressions to extract a maximum price introduced by under, below, up to, max, or maximum, and a size introduced by size. It removes these parts and an optional leading "looking for" phrase to get the search description.

**What moves through the session:** The query is parsed into `parsed`, and search output is saved in `search_results`. The first result becomes `selected_item`. The outfit tool reads `selected_item` and `wardrobe` from the session and saves its response in `outfit_suggestion`. The caption tool reads the saved outfit and item, then saves its response in `fit_card`. If search is empty, `error` holds useful advice and the loop stops.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30, size M'

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here are two outfit combinations featuring the **Y2K Baby Tee — Butterfly Print**:

### Outfit 1: Classic Y2K Streetwear
* **Bottoms:** Baggy straight-leg jeans, dark wash (`w_001`)
* **Shoes:** Chunky white sneakers (`w_007`)
* **Outerwear (Optional):** Vintage black denim jacket (`w_006`)
* **Why it works:** The fitted, cropped silhouette of the baby tee balances thevolume of the high-waisted, baggy dark-wash jeans for a classic Y2K proportion play. The white sneakers tie in the white base of the tee, while the black denimjacket adds an extra layer that fits the vintage aesthetic.

### Outfit 2: Casual Contrast
* **Bottoms:** Wide-leg khaki trousers (`w_002`)
* **Shoes:** Chunky white sneakers (`w_007`)
* **Accessories:** Brown leather belt (`w_009`)
* **Why it works:** Pairing the playful, pink-and-purple butterfly graphic tee with structured khaki trousers creates a fun mix of casual Y2K and minimal earthtones. Adding the brown leather belt pulls the tan trousers together while grounding the lighter top and shoes.

  Fit card: This Y2K white, pink, and purple butterfly print baby tee is available on Depop for $18. Styled with baggy dark-wash jeans and chunky white sneakers, it leans into a classic early 2000s streetwear vibe.

0 model calls this session, 2 served from cache
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print([(item['id'], item['size'], item['price']) for item in search_listings('graphic tee', size='M', max_price=30)])"
[('lst_002', 'S/M', 18.0), ('lst_017', 'S/M', 15.0)]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Here are two ways to style the vintage Levi's 501 jeans using pieces from your wardrobe:

### Outfit 1: Casual Streetwear
* **Top:** White ribbed tank top (`w_003`)
* **Outerwear:** Vintage black denim jacket (`w_006`)
* **Shoes:** Chunky white sneakers (`w_007`)
* **Accessories:** Black crossbody bag (`w_010`)

**Why it works:**
The medium-wash blue of the Levi's creates a classic high-contrast base againstthe white fitted tank and sneakers, keeping the look anchored in clean streetwear staples. Layering the slightly cropped black denim jacket ties in with the black crossbody bag while playing with denim-on-denim textures.

### Outfit 2: Cozy & Relaxed
* **Top:** Oversized grey crewneck sweatshirt (`w_004`)
* **Shoes:** Black combat boots (`w_008`)
* **Accessories:** Brown leather belt (`w_009`)

**Why it works:**
The fitted, straight-leg cut of the vintage 501s balances out the volume of theoversized grey crewneck. Tucking in the front of the sweatshirt and adding the brown leather belt pulls the casual look together, while the black combat boots give the outfit an easy, slightly grungier finish.
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
These vintage Levi's 501 jeans in a classic medium blue wash are listed on Depop for $38. They feature natural fading at the knees for an authentic, broken-in streetwear look. Pair them with simple white sneakers for an effortless everydayoutfit.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

**Moment 1**

- *What I asked for:* I asked Codex to help me specify and build the search tool step by step.
- *What came back:* The first specification used exact size matching and required every keyword. After reviewing tools.py, Codex identified that the starter expected ranked keyword matches and support for sizes such as S/M.
- *What I changed:* I updated the search specification and implemented keyword scoring, size filtering, and a maximum-price filter. I tested matching results, an empty search, and size M with a $30 budget.

**Moment 2**

- *What I asked for:* I asked Codex to help me connect the tools through session state and stop when a search finds nothing.
- *What came back:* Codex provided a planning loop that saves each tool's result in the session and checks search results before calling the outfit tool. It also provided commands to inspect the actual tool inputs and call counts.
- *What I changed:* I replaced the run_agent stub with the loop and tested both paths. The saved and received item IDs matched, and the empty-search test showed zero outfit calls, zero fit card calls, and a fit_card value of None.

**Part 2 — Testing and improvement**

- *What I asked for:* I asked Codex for step-by-step help moving search onto MCP, recording test evidence, and comparing the before and after results.
- *What came back:* Codex suggested an MCP wrapper, model-failure handlers, trace calls, and evaluation wrappers that recorded actual tool inputs. It also helped review the outputs and identified captions saying "Scored this" without a confirmed purchase.
- *What I changed:* I implemented and ran these steps, strengthened the caption prompt, and repeated the same evaluation with caching off. I documented the decrease in unsupported purchase claims from 2/25 to 0/25 and the remaining issue where an unknown brand was described as "unbranded".

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| 1. Matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Empty search stops before the outfit tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Saved item ID matches the outfit tool input | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card has accurate brand, color, and price details | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Search returns non-empty results within budget | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |


**Criterion 1 — Try 1**

Produced by `agent.py::run_agent`, recorded by `run_eval.py::run_once`. Caption generated by `tools.py::create_fit_card`.

```
tool_calls: [{'tool': 'search_listings', 'inputs': {'description': 'vintage graphic tee', 'size': None, 'max_price': 30.0}}, {'tool': 'suggest_outfit', 'item_id': 'lst_002'}, {'tool': 'create_fit_card', 'item_id': 'lst_002'}]
fit_card_is_none: False
```

Fit card:

```
Add this white, pink, and purple Y2K butterfly baby tee to your rotation for $18 on Depop. It features a fitted, cropped silhouette and comes in excellent condition. Pair it with baggy dark-wash jeans and chunky sneakers for an easy streetwear look that leans right into early 2000s proportions.
```

```
**Criterion 2 — Try 1**

Produced by `agent.py::run_agent`, recorded by `run_eval.py::run_once`.

```
stopped early: yes — No listings matched your search. Try different description keywords, another size, or a higher budget.
tool_calls: [{'tool': 'search_listings', 'inputs': {'description': 'designer ballgown', 'size': 'XXS', 'max_price': 5.0}}]
returned_item_prices: []
fit_card_is_none: True
```
```
**Criterion 3 — Try 1**

Session produced by `agent.py::run_agent`. Actual tool inputs captured by the observation wrappers in `run_eval.py::run_once`.

```text
selected_item_id: lst_002
tool_calls: [{'tool': 'search_listings', 'inputs': {'description': 'vintage graphic tee', 'size': 'M', 'max_price': 30.0}}, {'tool': 'suggest_outfit', 'item_id': 'lst_002'}, {'tool': 'create_fit_card', 'item_id': 'lst_002'}]
```

The saved item ID and the ID received by suggest_outfit both equal lst_002.

**Criterion 4 — Try 1**

Caption generated by `tools.py::create_fit_card`. Listing details and output recorded by `run_eval.py::run_once` and `run_eval.py::write_report`.

```
selected_item_brand: None
selected_item_colors: ['white', 'pink', 'purple']
selected_item_price: 18.0
```

Fit card:

```
Channel early 2000s style with this white, pink, and purple butterfly print baby tee, available on Depop for $18. This fitted crop top pairs effortlessly with baggy dark wash jeans and chunky white sneakers for a classic streetwear look. It comes in excellent condition with a Y2K vintage feel.
```

The caption uses the listed colors and price without inventing a brand.
**Criterion 5 — Try 1**

Results produced by `tools.py::search_listings` through MCP and recorded by `run_eval.py::write_report`.

```
parsed: {'description': 'graphic tee', 'size': None, 'max_price': 30.0}
returned_item_prices: [('lst_002', 18.0), ('lst_006', 24.0), ('lst_017', 15.0), ('lst_033', 19.0), ('lst_011', 27.0), ('lst_012', 20.0), ('lst_015', 26.0)]
```

The search returned seven items, and every price was at or below the $30 maximum.

Full evidence for all five tries per scenario: [Before run log](results/run_2026-10-11_1904_before.md).
---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Matching query completes all three tools | 4 of 5 | MET (5/5) | Every try recorded all three tool calls and returned a non-empty fit card. |
| 2 | Empty search stops before the outfit tool | 5 of 5 | MET (5/5) | Every try recorded only search, left fit_card as None, and suggested what to change. |
| 3 | Saved item ID matches the outfit tool input | 5 of 5 | MET (5/5) | The saved item ID and the actual suggest_outfit input ID were lst_002 in every try. |
| 4 | Fit card has accurate brand, color, and price details | 4 of 5 | MET (5/5) | All five captions in this scenario matched the listing's colors and $18 price and named no brand. |
| 5 | Search returns non-empty results within budget | 5 of 5 | MET (5/5) | Every try returned seven items, all priced at or below $30. |

**Diagnoses**

All five criteria met their original targets in the tested scenarios, so there are no criterion misses to diagnose. However, the tests were narrow: most matching scenarios selected the same butterfly tee, and the budget scenario repeated one price ceiling. These results do not prove reliability across all listings or query formats.

Criterion 4 checks brand, color, and price accuracy but misses other unsupported claims. In criterion 1, Try 5, the caption began "Scored this early 2000s butterfly print baby tee"; criterion 5, Try 3 also began "Scored this". This implies a purchase even though the user only searched for an item.

The issue occurs in the model output from tools.py::create_fit_card. Its prompt already says not to claim the user bought the item, but the model sometimes uses purchase language anyway. I will strengthen this one prompt and compare purchase claims before and after using the same scenarios.

For future testing, I would tighten criterion 4 to cover unsupported purchase claims as well as listing details. I am keeping the original criterion and target unchanged for this comparison.

---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```
```
$ python app.py ask 'vintage graphic tee under $30' --trace
[1] search_listings (via MCP)
      in:  {'description': 'vintage graphic tee', 'size': None, 'max_price': 30.0}
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    Matches found; select the first result.
[2] suggest_outfit
      in:  item_id=lst_002, wardrobe_ids=['w_001', 'w_002', 'w_003', 'w_004', 'w_005', 'w_006', 'w_007', 'w_008', 'w_009'…
      out: Here are two outfit combinations featuring the **Y2K Baby Tee —Butterfly Print**:  ### Outfit 1: Classic Y2K …
      →    Outfit saved in session; proceed to create_fit_card.
[3] create_fit_card
      in:  item_id=lst_002, outfit=Here are two outfit combinations featuring the **Y2K Baby Tee — Butterfly Print**:  ##…
      out: This Y2K white, pink, and purple butterfly print baby tee is available on Depop for $18. Styled with baggy dar…
      →    Fit card saved in session; run complete.
```
```

### Failure-mode checks

**Empty search**

```
$ python app.py ask 'designer ballgown size XXS under $5' --trace
[1] search_listings (via MCP)
      in:  {'description': 'designer ballgown', 'size': 'XXS', 'max_price': 5.0}
      out: [] (empty)
      →    Empty search; stop before outfit and fit card tools.

No listings matched your search. Try different description keywords, another size, or a higher budget.

0 model calls this session
```

```
$ python app.py ask 'designer ballgown size XXS under $5'

No listings matched your search. Try different description keywords, another size, or a higher budget.

0 model calls this session
```

The agent stopped and suggested changing the keywords, size, or budget.

```
**Empty wardrobe**

```
$ python app.py ask 'vintage graphic tee under $30' --empty-wardrobe
(running with an empty wardrobe)

Found: Y2K Baby Tee — Butterfly Print — $18.0 on depop

Outfit: ### Styling Advice for the Y2K Butterfly Baby Tee

This fitted, cropped baby tee in white with pink and purple graphics lendsitself well to nostalgic 2000s looks or mixed-aesthetic styling. Because it is form-fitting and short, balance the proportions with looser bottoms orlayer it for different textures.

**Model unavailable**

```
$ GEMINI_API_KEY=invalid-test-key AI201_CACHE=0 python app.py ask 'vintage graphic tee under $30'

The model was unavailable while generating outfit advice. Check your internet connection and GEMINI_API_KEY in .env, then try again.

1 model calls this session
```

I temporarily supplied an invalid key for this command and disabled caching to force a real model request. The agent caught ModelUnavailable and returned actionable advice instead of a traceback. The saved key in .env was unchanged.

### Suggested Complementary Pieces
* Bottoms: Low-rise baggy cargo pants, a denim mini skirt, or wide-leg track pants to lean into the Y2K aesthetic.
* Footwear: Chunky platform sandals, retro sneakers (like Nike Shox or retro runners), or platform mules.
* Outerwear: A cropped zip-up hoodie, a vintage leather jacket, or an unbuttoned mesh long-sleeve top worn underneath for a layered look.
* Accessories: A small shoulder bag (baguette style), butterfly clawclips, or beaded jelly jewelry.

Fit card: This Y2K white baby tee features a pink and purple butterfly graphic and a cropped, fitted silhouette. Listed on Depop for $18, it captures a nostalgic early 2000s mood. Pair it with low-rise baggy cargo pants and retro sneakers for an effortless throwback look.

2 model calls this session, 645 prompt + 254 output tokens
```

The agent returned general styling suggestions without claiming the user owned the suggested pieces.

```

**On the MCP move:** I registered search_listings in mcp_server.py and changed run_agent() in agent.py to call it through mcp_client.call_tool. The outfit and fit card tools still run directly. The query 'vintage graphic tee under $30' selected the same $18 Y2K Baby Tee and completed successfully after the change. The outfit and caption responses were served from cache.



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:** I strengthened the prompt in tools.py::create_fit_card to state that the user is browsing, with no confirmed purchase or ownership. It asks for listing language and suggested pairings, and explicitly excludes phrases such as "Scored this" and "I bought".

**Which failure it was meant to fix:** The baseline contained unsupported purchase claims in criterion 1, Try 5 and criterion 5, Try 3. I will compare all 25 generated captions before and after for claims that the user purchased, owns, or wore the item. This is an additional diagnostic measure; the five original criteria and their targets remain unchanged.

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Empty search stops before the outfit tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Saved item ID matches the outfit tool input | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card has accurate brand, color, and price details | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Search returns non-empty results within budget | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Did it help, and how do I know:**

The prompt change helped in this comparison. I reviewed all 25 generated captions in each run, including the empty-wardrobe diagnostic. Unsupported purchase, ownership, or actual-wearing claims decreased from 2/25 before to 0/25 after. The two baseline examples began "Scored this" in criterion 1, Try 5 and criterion 5, Try 3.

All five original criteria remained MET (5/5) in their assigned scenarios. The scenarios, model, temperature, and criteria stayed the same, and caching was off for both runs. Because model outputs vary and the sample is small, this is evidence of improvement in these runs, not a guarantee.

One issue remains: the after-run matching-query Try 1 described the item as "unbranded" even though its brand was unknown. The change addressed purchase claims, not every possible unsupported detail.

Full after-run evidence: [After run log](results/run_2026-10-11_1925_after.md).

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

No original criterion remained missed in its assigned test scenario, but the agent still has limitations.

In the after-run matching-query scenario, Try 1, the caption called the tee "unbranded" even though brand was None, meaning unknown. I would explicitly distinguish unknown from unbranded in the caption prompt and test listings with missing brands. I left this for future work to keep this unit's improvement limited to one measured change.

Search relevance is also imperfect: a graphic tee query can return pants or other items whose descriptions mention tees. Future work could improve keyword matching and test more clothing categories, sizes, and budgets.

Most matching scenarios selected the same butterfly tee. Broader testing is needed before claiming the results generalize to other listings. The purchase-claim improvement produced no such claims in 25 after-run captions, but that does not guarantee they will never occur.



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
