# Bakery Planner

Weekly and monthly production planner for the micro bakery, with order forms. It runs on the same Supabase project as Kitchen OS ("Silelis Kitchen"). The bakery data lives in its own `bk_` tables, and Kitchen OS is untouched.

## Files to upload
- `index.html`: the planner (you only, behind a login)
- `lab.html`: the Filling Lab, shown as its own section of the site (header: **Bakery Planner | Filling Lab**, address `#lab`)
- `order.html`: the order form. It works in two modes:
  - `order.html` for customers (online orders)
  - `order.html?staff=…` for colleagues (orders taken at the restaurant, with what was paid)

## One-time setup
1. **New GitHub repo** (e.g. `bakery`): upload `index.html`, `lab.html` and `order.html`, then turn on GitHub Pages (Settings → Pages → Deploy from branch → `main` / root). The address will look like `https://ordogas.github.io/bakery/`.
2. **Supabase → Authentication → URL Configuration:** set *Site URL* to that address, so confirmation and password-reset emails link back to the planner.
3. **Open the planner** and choose *First time? Create login* with `matas.gre@gmail.com`. Confirm the email, then sign in. Only emails listed in `bk_admins` get in. Anyone else who signs up sees nothing.
4. Optional: once your login works, turn off *Allow new users to sign up* (Authentication → Providers → Email). Kitchen OS doesn't use logins, so nothing else changes.
5. **Setup → Settings:** add the bank transfer recipient and IBAN, then copy the staff link for your colleagues and the online link for your website or Instagram.

## How it works
- **Products** are made of **recipes**. Each recipe is a chain of days, and its last day is when it's ready. A product sold on day D needs each recipe ready on D (or 1–3 days before, set per product). Steps are placed backwards from that day. For example, a 3-day focaccia sold on Monday is mixed on Saturday.
- **Shared recipes merge.** Sweet and savoury focaccia, or three bun types, become one dough card showing the total weight. If the recipe has a yield in grams, the card's ingredients are scaled to the batch.
- **Week template** → *Fill from template* adds the regular lineup for each day. You then change single days by tapping "On sale".
- **Open choices** (toppings, fillings, danish flavours) show as cards in *Needs attention* until decided, grouped per product per week. Ideas you save show up as quick picks next time.
- **Orders:** both forms only create *requests*. Approving one puts the order's recipe chain and a pickup card on the calendar. Changing the pickup date moves everything with it.
- **Layout:** the header switches between two sections, *Bakery Planner* and *Filling Lab*. In the planner, *Today / Week / Month* sit on the left, and *Orders / Stock / Sales / Setup* on the right.
- **Lab:** the Filling Lab, as its own section. "Use in planner" buttons on recipes, combos, trios, the pairing finder and the generator do two things:
  - Inside a decision ("Find it in the Lab"), the idea goes straight back into that decision.
  - Anywhere else, it's saved to your ideas list.
- **Recipe colours:** set a colour per recipe in Setup → Recipes. Cards and day dots use it, and products show it next to their name.
- **Stock:** for pastries made in batches and sold over several days (almond cookies, tinginys), tick *Made in batches and sold from stock* on the product.
  - Daily plan quantities then count as expected sales, and no daily bake steps are generated.
  - The Stock tab shows what's on hand and a 14-day forecast. When stock runs short, a card says when to make the next batch.
  - A planned batch adds its recipe steps to the calendar. Ticking the last step done adds the batch to what's on hand, and end-of-day sales and waste come off it.
- **Freezer prep** (also in the Stock tab): set a minimum stock level for each item. When stock drops below it, a restock card appears on the week board.
- **Sales:** a daily log per item of made / sold / discounted / sold-out time, which saves as you type. There's a "Now" button for sold out, and you can go back to earlier days.
  - Tapping the "left" count sorts leftovers into **carry over** (shown as "+N from yesterday" the next day), **reused** (casserole, croutons, pangrattato), **given away** or **binned**.
  - For stock products, unsold pieces stay in stock, so only reused, given away and binned apply.
  - Below the log, *How items are selling* covers the last 7 days, 4 weeks or the season. Summary tiles show sold, carried over, reused, given away and binned. Per item it shows sold, per day, sell-through, leftovers with their breakdown, sold-out count and time, and a weekday chart against your template.
  - It suggests template changes where an item keeps selling out or has over 15% left over, and *Update template* applies them.
  - Sales of stock products come off their on-hand count.

## Security
- RLS limits every `bk_` table to the confirmed admin login.
- The forms can only call two functions: `bk_public_form`, which reads orderable products and form texts, and `bk_submit_order`, which inserts a request and checks the minimum notice.
- Customer details can't be read back through the public key.
- The staff link carries a token, which you can regenerate in Settings.
- **Kitchen OS tables are still open to anyone with the key.** Locking them down is a separate job.

## Editing the code
`src/` holds the sources. `lab-source.html` is a copy of the Filling Lab and `lab-bridge.js` connects it to the planner. Run `python3 build.py` to rebuild `dist/`. `sql/` holds the schema and starter data (already applied). `test/` runs headless tests against an in-memory database.
