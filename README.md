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

This is a tool where a user can ask for a specific item and get outfit suggestions along with a capiton. One of the 3 tools is used to find listings matching to the item and then is used for outfit suggestions in the next tool. Lastly, the user will get a descriptive caption about the piece and its style. 

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** searches for items matching the descriptions  
- **Inputs:** description (str), size (str), max_price (float or none)
- **Returns:** returns a list of items matching the descriptions 
- **When it has nothing:** returns empty list 

### `suggest_outfit`

- **What it does:** generates outfits based on wardrobe 
- **Inputs:** (dict) new_item - a single listing from search_listings, wardrobe (dict) - items key holding list 
- **Returns:** returns str with at least one outfit suggestion 
- **When it has nothing:** returns general styling advice rather raising an error

### `create_fit_card`

- **What it does:** writes a short caption
- **Inputs:** (str) outfit - suggestion string from suggest_outfit, (dict) new_item - the selected listing
- **Returns:** returns a generated caption (str), 2-4 sentence caption 
- **When it has nothing:** if outfit is empty, nothing would be outputted as the program would be stopped. otherwise returns a descriptive caption 

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

**Branch rule:** if search_listings returns an empty, put a message in the session that will name the query and filters used (e.g. no listings under $5 in size XXS) and stop with no further tool calls. otherwise take the first result and go to the next tool. 

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** with regex, in `agent.py::parse_query`. It makes no model call. One pattern finds the price ceiling ("under $30", "below 40", "up to $25", or a bare "$30") and another finds the size ("size M", "in size 8", "size US 9"). Each match is cut out of the text, and what remains, with commas removed, becomes the description. A size or price that isn't in the query comes back as `None`, which tells `search_listings` to skip that filter. Example: "vintage graphic tee under $30, size M" becomes description "vintage graphic tee", size "M", max_price 30.0.

**What moves through the session:** each step writes its result into the session, and the next step reads it back out instead of receiving it directly. In order:

1. `query` — what the user typed, set when the session starts.
2. `parsed` — description, size and max_price from `parse_query`.
3. `search_results` and `searched` — everything `search_listings` returned, and a flag that the search has run. If the list is empty, `error` is set to a message naming the query and the filters used, and the loop stops here.
4. `selected_item` — the first search result.
5. `outfit_suggestion` — what `suggest_outfit` returned for `selected_item` and `wardrobe`.
6. `fit_card` — what `create_fit_card` returned for the outfit and the item.

`wardrobe` is set at the start and only read. `error` stays `None` unless the run ends early, and in that case `selected_item`, `outfit_suggestion` and `fit_card` are all still `None`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask '...'

 python app.py ask
Ask for something, or press Enter on an empty line to quit.

> levis jeans

  Found:    Vintage Levi's 501 Jeans — Medium Wash — $38.0 on depop

  Outfit:   **Outfit 1: Casual Streetwear**
*   **New Find:** Vintage Levi's 501 Jeans — Medium Wash
*   **Top:** White ribbed tank top
*   **Outerwear:** Vintage black denim jacket 
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Outfit 2: Cozy Layered Look**
*   **New Find:** Vintage Levi's 501 Jeans — Medium Wash
*   **Top:** Oversized grey crewneck sweatshirt
*   **Shoes:** Black combat boots
*   **Accessories:** Brown leather belt

  Fit card: Nothing beats the wash on these vintage Levi's 501 jeans—the ultimate heavy-cotton staple your closet’s been begging for. Throw them on with a chunky sweater and boots for moody autumn strolls, or keep it breezy with a ribbed tank and sneakers. Grab this dream pair on my Depop right now for just $38.


```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

```
python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}, {'id': 'lst_012', 'title': 'Oversized Crewneck Sweatshirt — Vintage Navy', 'description': 'Perfectly faded navy crewneck. Genuinely vintage — not manufactured distressed. Ribbed cuffs and hem. No graphics, clean.', 'category': 'tops', 'style_tags': ['vintage', 'basics', 'oversized', 'classic'], 'size': 'XL (fits oversized)', 'condition': 'good', 'price': 20.0, 'colors': ['navy'], 'brand': None, 'platform': 'thredUp'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```
$ python -c "from tools import suggest_outfit; ..."

```
python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

**Outfit 1: Casual Streetwear**
*   **New Find:** Vintage Levi's 501 Jeans — Medium Wash
*   **Top:** White ribbed tank top
*   **Outerwear:** Vintage black denim jacket 
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Outfit 2: Cozy Layered Look**
*   **New Find:** Vintage Levi's 501 Jeans — Medium Wash
*   **Top:** Oversized grey crewneck sweatshirt
*   **Shoes:** Black combat boots
*   **Accessories:** Brown leather belt
(.venv) nikolai@MacBook-Air-5 ai201-project2-fitfindr-starter-v2026
```
$ python -c "from tools import create_fit_card; ..."

```
python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

Nothing beats finding a classic pair of vintage Levi's 501 jeans that already have that perfect, broken-in feel. I'm obsessed with this medium wash, and they're up now on my Depop for $38. Pair them with fresh white sneakers and you've got that effortlessly cool off-duty look locked down.
---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked for a specific item 
- *What came back:* As a result a listing came up with at least one matching keyword even if its a completely different item. 
- *What I changed:* I made sure every keyword from search matched the exact listing 
**Moment 2**

- *What I asked for:* I asked for Claude to help me write the loop.
- *What came back:* Claude wrote the loop but I had to make sure the selected_item in the search_listings was the same as in suggest_outfit
- *What I changed:* Claude initially didn't take that into consideration so I made sure sure that the item was consistent throughout 

**Moment 3**
I used Claude to help me figure out which criteria I should refine so I decided with the 3rd one. 

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
| 1. A query the data can match. | 4 of 5 | PASS | PASS | PASS | PASS | PASS | 5/5 |
| 2. A query nothing can match | 5 of 5| PASS | PASS| PASS| PASS | PASS | 5/5 |
| 3. Item in new_item and search_item must match | 5 of 5 | PASS  | PASS | PASS | PASS | PASS | 5/5 |
| 4. No two fit-card captions open with same first three words| 4 of 5 | PASS | PASS | PASS | PASS | PASS | 5 of 5 |
| 5.  Every returned listing has price under max_price | 5 of 5 | PASS | PASS | PASS| PASS | PASS | 5/5 |

**Real output from one try**, pasted as text, naming the file and function

file - results/run_2026-10-13_1336_before.md
that produced it:

```

agent.py, run_agent
**Try 1**

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 3

Outfit suggestion:

```
**Outfit 1: Y2K Streetwear**
*   Y2K Baby Tee — Butterfly Print (New Find)
*   Baggy straight-leg jeans, dark wash
*   Chunky white sneakers
*   Black crossbody bag

**Outfit 2: Edgy Contrast**
*   Y2K Baby Tee — Butterfly Print (New Find)
*   Wide-leg khaki trousers
*   Black combat boots
*   Vintage black denim jacket
```

- Query: `denim jacket under $50`
- Wardrobe: empty

agent.py, run_agent
**Try 1**

- stopped early: no
- selected_item: Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
- search_results: 1

Outfit suggestion:

```
Here is how to style that cropped light wash denim jacket:

**Look 1: Casual Streetwear**
*   **The cropped light wash denim jacket (new find)** balanced with an oversized, black graphic band tee.
*   Pair with olive green cargo pants and chunky black combat boots. 
*   Finish with a canvas crossbody bag.

**Look 2: Elevated Contrast**
*   **The cropped light wash denim jacket (new find)** layered over a fitted black ribbed turtleneck.
*   Pair with a plaid pleated midi skirt and knee-high leather riding boots.
*   Add a structured leather shoulder bag to dress it up.
```

Fit card:

```
Nothing beats a good cropped Wrangler denim jacket for throwing over a band tee or dressing up with a pleated midi skirt. Found this gem on Poshmark for $42 and honestly, it goes with literally everything in my closet. It’s got that perfectly broken-in 90s vibe without any of the hunting.
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 1 items: Denim Jacket — Light Wash, Cropped
[2] suggest_outfit
      in:  Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
      out: Here is how to style that cropped light wash denim jacket:  **Look 1: Casual Streetwear** *   **The cropped li…
[3] create_fit_card
      in:  Here is how to style that cropped light wash denim jacket:  **Look 1: Casual Streetwear** *   **The cropped li…
      out: Nothing beats a good cropped Wrangler denim jacket for throwing over a band tee or dressing up with a pleated …
```



Fit card:

```
Little butterfly baby tee just landed on the Depop for $18 and she is giving total early 2000s mall rat energy. I’m obsessed with pairing her with baggy denim and chunky sneaks, but she looks just as good toughened up with combat boots and a worn-in jacket. Grab her before I change my mind and keep it for myself!
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 3 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey
[2] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: **Outfit 1: Y2K Streetwear** *   Y2K Baby Tee — Butterfly Print (New Find) *   Baggy straight-leg jeans, dark …
[3] create_fit_card
      in:  **Outfit 1: Y2K Streetwear** *   Y2K Baby Tee — Butterfly Print (New Find) *   Baggy straight-leg jeans, dark …
      out: Little butterfly baby tee just landed on the Depop for $18 and she is giving total early 2000s mall rat energy…
```

agent.py, run_agent 
**Try 1** 

- stopped early: no
- selected_item: Vintage Levi's 501 Jeans — Medium Wash ($38.0, depop)
- search_results: 3

Outfit suggestion:

```
**Outfit 1: Casual Streetwear**
*   Vintage Levi's 501 Jeans — Medium Wash (New Find)
*   White ribbed tank top
*   Black cropped zip hoodie
*   Chunky white sneakers
*   Black crossbody bag

**Outfit 2: Cozy Layered**
*   Vintage Levi's 501 Jeans — Medium Wash (New Find)
*   Oversized grey crewneck sweatshirt
*   Black combat boots
*   Brown leather belt
```

Fit card:

```
The search for the perfect vintage Levi's 501 is officially over. These medium wash denim legends have that heavy, broken-in feel and are just sitting on my Depop for $38.00. Whether you're throwing them on with a zip-up and chunky sneakers or slinging a belt over them with an oversized crewneck, they do all the heavy lifting.
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 3 items: Vintage Levi's 501 Jeans — Medium Wash, Baggy Carpenter Jeans — Dark Wash, Straight Leg Black Jeans — Faded
[2] suggest_outfit
      in:  Vintage Levi's 501 Jeans — Medium Wash ($38.0, depop)
      out: **Outfit 1: Casual Streetwear** *   Vintage Levi's 501 Jeans — Medium Wash (New Find) *   White ribbed tank to…
[3] create_fit_card
      in:  **Outfit 1: Casual Streetwear** *   Vintage Levi's 501 Jeans — Medium Wash (New Find) *   White ribbed tank to…
      out: The search for the perfect vintage Levi's 501 is officially over. These medium wash denim legends have that he…
```

- Query: `vintage graphic tee under $30`
- Wardrobe: example

agent.py, run_agent
**Try 1**

- stopped early: no
- selected_item: Y2K Baby Tee — Butterfly Print ($18.0, depop)
- search_results: 3

Outfit suggestion:

```
**Outfit 1: Y2K Streetwear**
*   Y2K Baby Tee — Butterfly Print (New find)
*   Baggy straight-leg jeans, dark wash
*   Chunky white sneakers
*   Black crossbody bag

**Outfit 2: Edgy Contrast**
*   Y2K Baby Tee — Butterfly Print (New find)
*   Wide-leg khaki trousers
*   Black combat boots
*   Vintage black denim jacket
```

Fit card:

```
Found this butterfly print Y2K baby tee for just $18 and I am obsessed. It's giving off-duty pop star paired with baggy denim and chunky sneakers, but I love it just as much toughened up with combat boots and trousers. Up now on my depop!
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 3 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey
[2] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: **Outfit 1: Y2K Streetwear** *   Y2K Baby Tee — Butterfly Print (New find) *   Baggy straight-leg jeans, dark …
[3] create_fit_card
      in:  **Outfit 1: Y2K Streetwear** *   Y2K Baby Tee — Butterfly Print (New find) *   Baggy straight-leg jeans, dark …
      out: Found this butterfly print Y2K baby tee for just $18 and I am obsessed. It's giving off-duty pop star paired w…
```

tools.py, search_listings

**Try 1**

- stopped early: no
- selected_item: High-Waisted Denim Shorts — Cutoff ($24.0, poshmark)
- search_results: 1

Outfit suggestion:

```
**Outfit 1: Casual Summer Streetwear**
* High-Waisted Denim Shorts — Cutoff (New Find)
* White ribbed tank top
* Black cropped zip hoodie
* Chunky white sneakers
* Black crossbody bag

**Outfit 2: Edgy Grunge Layering**
* High-Waisted Denim Shorts — Cutoff (New Find)
* White ribbed tank top
* Vintage black denim jacket
* Black combat boots
* Brown leather belt
```

Fit card:

```
Finally tracked down the holy grail of vintage denim. These Levi's cutoff shorts were an absolute steal on Poshmark for just $24. They have that perfectly broken-in, effortless 90s slouch whether you're throwing them on with chunky sneakers for the weekend or toughening them up with combat boots. Tap the link to shop before I change my mind and keep them.
```

Trace:

```
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 1 items: High-Waisted Denim Shorts — Cutoff
[2] suggest_outfit
      in:  High-Waisted Denim Shorts — Cutoff ($24.0, poshmark)
      out: **Outfit 1: Casual Summer Streetwear** * High-Waisted Denim Shorts — Cutoff (New Find) * White ribbed tank top…
[3] create_fit_card
      in:  **Outfit 1: Casual Summer Streetwear** * High-Waisted Denim Shorts — Cutoff (New Find) * White ribbed tank top…
      out: Finally tracked down the holy grail of vintage denim. These Levi's cutoff shorts were an absolute steal on Pos…
```



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
| 1 | A query the data can match.  | 5/5| met | for 5 of 5 runs the data matched the query  |
| 2 | A query nothing can match | 5/5 | met | for each run where nothing was found the program stopped |
| 3 | Item in new_item and search_item must match | 5/5  | met | 
for each try the item from listing and in search_item matched |
| 4 | No two fit-card captions open with same first three words | 4/5 | met | each fit_card must begin with different sentence listing |
| 5 | Every returned listing has price under max_price | 5/5 | met | what was returned was in price range |

**Diagnoses**



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
python app.py ask 'vintage graphic tee under $30' --trace                  

[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 3 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey
[2] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: **Outfit 1: Y2K Streetwear** *   Y2K Baby Tee — Butterfly Print (New find) *   Baggy straight-leg jeans, dark …
[3] create_fit_card
      in:  **Outfit 1: Y2K Streetwear** *   Y2K Baby Tee — Butterfly Print (New find) *   Baggy straight-leg jeans, dark …
      out: Found the ultimate Y2K baby tee with the dreamiest butterfly print. It's giving 2000s mall rat whether you're …

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   **Outfit 1: Y2K Streetwear**
*   Y2K Baby Tee — Butterfly Print (New find)
*   Baggy straight-leg jeans, dark wash
*   Chunky white sneakers
*   Black crossbody bag

**Outfit 2: Edgy Contrast**
*   Y2K Baby Tee — Butterfly Print (New find)
*   Wide-leg khaki trousers
*   Black combat boots
*   Vintage black denim jacket

  Fit card: Found the ultimate Y2K baby tee with the dreamiest butterfly print. It's giving 2000s mall rat whether you're throwing it on with baggy denim and chunky sneakers or toughening it up with combat boots and khakis. Grab this little vintage gem on Depop right now for just $18.00 before I keep it for myself.

0 model calls this session, 2 served from cache
```

**Empty search**

```
python app.py ask 'pink jeans under $30' --empty-wardrobe --trace          

(running with an empty wardrobe)
[1] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: [] (empty)
[2] stop
      →    branch: search returned nothing, no further tool calls

  No listings matched "pink jeans" (under $30) for the query: 'pink jeans under $30'. You could raise your price limit or try different keywords.

0 model calls this session

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->


**What I changed:**
Something I changed is criteria 4 to target the first 3 words of the fit_card caption rather than the whole sentence. 
 
**Which failure it was meant to fix:**
I didn't miss any of my targets this time as well so I just had to refine my criteria 4. 

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |   A query the data can match.| 5/5 | 5/5 | 5/5| 5/5| 5/5| pass|
| 2.  |  A query nothing can match| 5/5 | 5/5 | 5/5 | 5/5 | 5/5 | pass |
| 3.  |  Item in new_item and search_item must match| 5/5 | 5/5 | 5/5 | 5/5 | 5/5 | pass |
| 4.  |  No two fit-card captions open with same first three words| 5/5 | 5/5 | 5/5 | 5/5 | 5/5 | pass|
| 5.  | Every returned listing has price under max_price  | 5/5 | 5/5 | 5/5 | 5/5 | 5/5| pass |


**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->
Yes, refining my criteria did help. Under my original wording, criterion 4 passed in every try because no two captions had identical first sentence. When I changed it to compare the first three words, I was closer to missing the target. 




---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->

There is nothing broken however, I could be more precise with my targets next time. 
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
