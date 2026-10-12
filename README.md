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
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
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

<!-- Three or four sentences: what a user asks for, and what they get back. -->
FitFindr is an agent that helps a user shop thrift listings. The user asks for
an item in plain language, such as `vintage graphic tee under $30`, and the
agent searches the listings data for matches. If it finds one, it suggests how
to style the item with the user's wardrobe and writes a short fit-card caption.
If it finds nothing, it stops early and tells the user what to change.


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

- **What it does:** Searches `data/listings.json` for thrift listings that match the user's description, optional size, and optional max price.
- **Inputs:** `description` (`str`), `size` (`str | None`), `max_price` (`float | None`).
- **Returns:** A list of listing dictionaries, sorted by keyword match score, each with fields like `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.
- **When it has nothing:** Returns an empty list `[]`.

### `suggest_outfit`

- **What it does:** Suggests how to style the selected listing with the user's saved wardrobe.
- **Inputs:** `new_item` (`dict` listing), `wardrobe` (`dict` with an `items` list).
- **Returns:** A non-empty outfit suggestion string that names the selected item and, when possible, wardrobe pieces that go with it.
- **When it has nothing:** If the wardrobe is empty, returns general styling advice for the item instead of failing.

### `create_fit_card`

- **What it does:** Writes a short social caption for the selected item and outfit idea.
- **Inputs:** `outfit` (`str`), `new_item` (`dict` listing).
- **Returns:** A two-to-four sentence caption mentioning the item, price, platform, and vibe.
- **When it has nothing:** If `outfit` is blank, returns a message saying a fit card cannot be created without an outfit suggestion.

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

**Branch rule:**
If `search_listings` returns an empty list, put a helpful message in
`session["error"]` and stop before calling `suggest_outfit`. Otherwise, save the
first listing in `session["selected_item"]`, pass that session item into
`suggest_outfit`, then pass the resulting outfit and same session item into
`create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** <!-- regex, string splitting, or asking the model — say which -->
Regex and string cleanup. The loop extracts `under $N` as `max_price`, `size X`
as `size`, and uses the remaining words as the search description.

**What moves through the session:** <!-- which fields, in what order -->
The parsed query goes into `session["parsed"]`, search results go into
`session["search_results"]`, the first result goes into
`session["selected_item"]`, the outfit text goes into
`session["outfit_suggestion"]`, and the final caption goes into
`session["fit_card"]`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30, size M'

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here are two Y2K-inspired outfits featuring your new **lst_002**:

**Outfit 1: Y2K Streetwear**
Pair the baby tee with your baggy straight-leg dark wash jeans for a classic early 2000s silhouette. Layer the black cropped zip hoodie on top for easy contrast, and finish with chunky white sneakers and the black crossbody bag for an effortless, everyday look.

**Outfit 2: Casual Edge**
Tuck the butterfly tee into your wide-leg khaki trousers, accented by the brown leather belt to tie the look together. Throw on the vintage black denim jacket for a touch of grunge, and step into chunky white sneakers to keep the outfit fresh, comfortable, and balanced.

  Fit card: Channeling major early 2000s energy with this dreamy butterfly baby tee! It’s giving effortless streetwear and fits like a dream (S/M). Grab it over on my Depop right now for just $18 before someone else snatches it up!

```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print([(x['id'], x['title'], x['price'], x['size']) for x in search_listings('graphic tee', max_price=30)])"
[('lst_017', 'Mesh Long-Sleeve Top — Black', 15.0, 'S/M'), ('lst_002', 'Y2K Baby Tee — Butterfly Print', 18.0, 'S/M'), ('lst_033', 'Vintage Band Tee — Faded Grey', 19.0, 'L'), ('lst_006', 'Graphic Tee — 2003 Tour Bootleg Style', 24.0, 'L')]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[1], get_example_wardrobe()))"
Here are two Y2K-inspired outfits featuring your new **lst_002**:

**Outfit 1: Y2K Streetwear**
Pair the baby tee with your baggy straight-leg dark wash jeans for a classic early 2000s silhouette. Layer the black cropped zip hoodie on top for easy contrast, and finish with chunky white sneakers and the black crossbody bag for an effortless, everyday look.

**Outfit 2: Casual Edge**
Tuck the butterfly tee into your wide-leg khaki trousers, accented by the brown leather belt to tie the look together. Throw on the vintage black denim jacket for a touch of grunge, and step into chunky white sneakers to keep the outfit fresh, comfortable, and balanced.
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('Pair it with baggy jeans and white sneakers.', load_listings()[1]))"
Obsessed with this Y2K baby tee with the cutest butterfly print! I styled it with some baggy jeans and white sneakers for the ultimate casual fit. Grab it now for just $18.0 over on depop before it’s gone!

```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked AI to help explain the FitFindr assignment and separate it from the previous RAG project.
- *What came back:* It identified that this is a new Project 2 repo with tools, a planning loop, session state, criteria, and a README submission.
- *What I changed:* I created a separate FitFindr project folder instead of mixing the work into `ai201-project1`.

**Moment 2**

- *What I asked for:* I asked AI to help pressure-test the search and loop design while implementing the tools.
- *What came back:* It caught the PowerShell `$30` quoting issue and pointed out that weak one-word matches could return unrelated listings.
- *What I changed:* I used single-quoted app commands, added simple query parsing, ignored filler words in search, and required stronger keyword overlap for multi-word searches.

**Moment 3**

- *What I asked for:* I asked AI to turn the testing brief into a checklist for MCP, failure modes, five-run evaluation, and the one measured improvement.
- *What came back:* It identified the missing criterion scenarios and recommended recording actual run output rather than treating one successful run as evidence.
- *What I changed:* I added one scenario per remaining criterion, ran the required before and after evaluations with caching disabled, and recorded the exact handler messages for the three failure modes.

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

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Impossible query stops before tool 2 | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item id carries into outfit text | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card includes item, price, and platform | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Search respects the price ceiling | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```
`results/run_2026-10-12_0929_before.md`, produced by `run_eval.py::main` and
`agent.py::run_agent`, contains all 25 criterion tries (plus the separate
empty-wardrobe diagnostic). Representative real output from one try per
criterion follows.

Criterion 1:

    - stopped early: no
    - selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
    - search_results: 5
    Fit card: Channeling major 2000s energy with this dreamy butterfly baby tee, now up on my depop for just $18!

Criterion 2:

    [2] search_listings (via MCP)
          out: [] (empty)
    [3] branch
          out: I could not find a matching listing. Try a broader description, a different size, or a higher max price.
          →    empty search, stopping

Criterion 3:

    - selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
    Outfit suggestion: Embrace a Y2K aesthetic by pairing your **lst_002** butterfly baby tee with the baggy straight-leg dark wash jeans.

Criterion 4:

    Fit card: Obsessed with this Y2K butterfly baby tee and honestly might keep it for myself! It’s up on Depop right now for just $18.0, perfect for styling with baggy denim or edgy cargo pants.

Criterion 5:

    - selected_item: Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
    - search_results: 1
```

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
| 1 | Matching query completes all three tools | 4 of 5 | MET (5/5) | Each run reached `create_fit_card` and returned a non-empty fit card. |
| 2 | Impossible query stops before tool 2 | 5 of 5 | MET (5/5) | Every trace ended after the empty `search_listings` result; no outfit suggestion was present. |
| 3 | Selected item id carries into outfit text | 4 of 5 | MET (5/5) | The selected id was `lst_002` and each outfit suggestion included `lst_002`. |
| 4 | Fit card includes item, price, and platform | 4 of 5 | MET (5/5) | Each card named the baby tee, $18/$18.0, and Depop/depop. |
| 5 | Search respects the price ceiling | 5 of 5 | MET (5/5) | The $50 scenario returned one $42.0 listing on every try; no result exceeded its parsed ceiling. |

**Diagnoses**

There were no missed criteria in the before run, so there is no failure to
diagnose among the five scored criteria. The likely weak point was criterion 4:
the old fit-card prompt asked for the right fields but left their exact wording
to model paraphrase. I chose that as the one improvement target.

I also triggered the required failure modes separately:

- Empty search: “I could not find a matching listing. Try a broader description, a different size, or a higher max price.”
- Empty wardrobe: “Building your first wardrobe around lst_007 is easy because light-wash denim goes with everything. Here are two practical outfits:” The agent then supplied general styling advice rather than crashing.
- Model unavailable: “The model rejected your API key. Check GEMINI_API_KEY in your .env file, or create a fresh key at aistudio.google.com.” I tested this with a temporary invalid environment value and restored the normal configuration immediately afterward.


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
$ python app.py ask 'vintage graphic tee under $30' --trace
[1] parse_query
      in:  vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 5 items: Y2K Baby Tee — Butterfly Print, Vintage Band Tee — Faded Grey, Graphic Tee — 2003 Tour Bootleg Style … +2 more
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] suggest_outfit
      in:  dict with keys: new_item, wardrobe_items
      out: Here are two Y2K-inspired outfits featuring your new **lst_002**:  **Outfit 1: Y2K Streetwear** Pair the baby …
[5] create_fit_card
      in:  dict with keys: outfit, new_item
      out: Channeling major early 2000s energy with this dreamy butterfly baby tee! It’s giving effortless streetwear and…
```

**Empty search**

```
$ python app.py ask 'designer ballgown size XXS under $5' --trace
[1] parse_query
      in:  designer ballgown size XXS under $5
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
[3] branch
      out: I could not find a matching listing. Try a broader description, a different size, or a higher max price.
      →    empty search, stopping
```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->
I moved `search_listings` from a direct Python call to `mcp_client.call_tool`.
The happy path and empty-search path behaved the same after the move: MCP
changed the call shape, but the result still came back as the same list of
listing dictionaries.


---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

I changed only the `create_fit_card` prompt in `tools.py`. Instead of asking
the model generally to mention the item, price, and platform, it now gives the
exact title, price, and platform strings and requires each once.

**Which failure it was meant to fix:**

This hardens criterion 4 against a model paraphrase that might omit or alter a
listing detail. The before run happened to pass, but the prompt contract was
not as explicit as the criterion.

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Impossible query stops before tool 2 | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item id carries into outfit text | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card includes item, price, and platform | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Search respects the price ceiling | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

**Representative after-run output** from
`results/run_2026-10-12_0932_after.md` (`run_eval.py::main` and
`agent.py::run_agent`):

```
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
Outfit suggestion: Here are two outfits featuring your **lst_002**.
Fit card: Living out my early 2000s dreams in this adorable Y2K Baby Tee — Butterfly Print! ... Grab this little beauty on my depop right now for just $18.0 before someone else does!

[2] search_listings (via MCP)
      out: [] (empty)
[3] branch
      out: I could not find a matching listing. Try a broader description, a different size, or a higher max price.

- selected_item: Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
- search_results: 1
```

**Did it help, and how do I know:**

`results/run_2026-10-12_0932_after.md` records the after run. It again passed
criterion 4 in 5/5 tries, and the cards now included the exact listing title,
price, and platform. The overall numeric score did not increase because the
before run was already 5/5; this change made the behavior more explicit rather
than producing a higher count.

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken
No scored criterion was missed after the improvement. The remaining limitation
is that generated caption style still varies, and prompt instructions alone
cannot guarantee every future model response will comply perfectly. I stopped
after one prompt change because the assignment requires one measured change;
the next production step would be a post-generation validator with a retry,
which would be a separate system change.
<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



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
