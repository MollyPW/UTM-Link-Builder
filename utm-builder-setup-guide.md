# UTM Builder — Customization & Team Setup Guide

This doc covers two separate things:

1. **Customizing** the tool (default values, labels, colors) — no coding experience needed, just editing text in the file.
2. **Sharing it with your team** so everyone sees the same saved options and presets, by connecting a free shared database (Supabase).

Do Part 1 first if you want it. Part 2 is optional and only needed if multiple people need to see each other's saved values.

---

## Part 1: Customizing the tool

Open `utm-builder.html` in any plain text editor (Notepad, TextEdit, VS Code — anything works, just not Word).

### 1.1 Change the starting dropdown values

Find this block near the top of the `<script>` section:

```js
const DEFAULT_OPTIONS = {
  source:   ['newsletter','facebook','linkedin','google','twitter','partner-site','website'],
  medium:   ['email','social','cpc','organic','referral','display','affiliate'],
  campaign: ['spring-sale','product-launch','landing-page-launch','webinar-2026'],
  content:  ['header-cta','body-link-1','body-link-2','footer-link','sidebar-banner','ps-line'],
  term:     []
};
```

**To edit:** change the words inside the quotes. Keep the format `'word-here'` with commas between values, and lowercase-with-hyphens (no spaces).

> Note: this only sets the *starting* list. Once someone opens the tool, their own added/removed values take over (stored in their browser). If you want to reset everyone back to these defaults, see step 1.3.

### 1.2 Change the starting presets

Just below that, find:

```js
const DEFAULT_PRESETS = [
  {name:'New landing page', source:'website', medium:'referral', campaign:'landing-page-launch', content:'', term:''},
  {name:'Email newsletter',  source:'newsletter', medium:'email', campaign:'', content:'header-cta', term:''}
];
```

**To edit:** change `name` to whatever you want the preset called, and fill in whichever fields that preset should auto-fill. Leave a field as `''` (empty) if that preset shouldn't touch it.

**To add a new preset**, copy one whole line, paste it as a new line inside the brackets, and edit the values.

### 1.3 Change the title, colors, or field labels

- Page title: edit the `<h1>UTM Link Builder</h1>` line and the `<title>` tag near the top.
- Colors: near the top of the `<style>` section, there's a block starting with `:root{`. Each line like `--accent:#2D5BFF;` is one color — replace the hex code with another one.
- Field labels (e.g. renaming "Content" to something your team prefers): find the `FIELDS` array near the top of the script, and edit the `label:'Content'` text.

Save the file and reopen it in your browser to see changes — no build step required.

---

## Part 2: Sharing with your team (adding a database backend)

Right now, everyone who opens this tool has their **own separate** saved options and presets (stored per-browser). This section connects the tool to **Supabase**, a free hosted database, so everyone using the same copy of the tool sees the same shared values.

This takes about 15 minutes. You'll need: a web browser, and a free Supabase account (no credit card required for this).

### Step 1 — Create a Supabase project

1. Go to **supabase.com** and sign up (free tier is enough).
2. Click **New Project**.
3. Give it a name (e.g. `utm-builder`), set a database password (save it somewhere), pick any region.
4. Click **Create new project** and wait ~2 minutes for it to finish setting up.

### Step 2 — Create the data table

1. In your new project, click **SQL Editor** in the left sidebar.
2. Click **New query**.
3. Paste in this exact code:

```sql
create table utm_builder_data (
  key text primary key,
  value jsonb not null,
  updated_at timestamptz default now()
);

alter table utm_builder_data enable row level security;

create policy "Allow anon read" on utm_builder_data
  for select using (true);

create policy "Allow anon write" on utm_builder_data
  for insert with check (true);

create policy "Allow anon update" on utm_builder_data
  for update using (true);
```

4. Click **Run**.

**What this did:** created one table that stores your options list and presets list, and opened it up so the tool can read and write to it.

> ⚠️ **Security note:** these settings let *anyone with your Supabase URL and key* read and write this data — there's no login. That's fine for a small trusted team using an internal tool, but don't use this setup for anything sensitive, and don't publish the keys publicly beyond your team.

### Step 3 — Get your connection details

1. In the left sidebar, click the **gear icon (Project Settings)**, then **API**.
2. You'll see two values you need:
   - **Project URL** (looks like `https://abcdxyz.supabase.co`)
   - **anon public key** (a long string of letters/numbers)
3. Copy both somewhere handy — you'll paste them into the tool file next.

### Step 4 — Edit the tool file

Open `utm-builder.html` in a text editor and make these three changes:

**A. Add the Supabase library.** Find this line near the top of the `<head>` section:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
```

Add this line right above it:

```html
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
```

**B. Add your connection details.** Find this line:

```js
// ---------- Config ----------
```

Add these three lines right after it (paste in your own URL and key from Step 3):

```js
const SUPABASE_URL = 'PASTE-YOUR-PROJECT-URL-HERE';
const SUPABASE_ANON_KEY = 'PASTE-YOUR-ANON-KEY-HERE';
const supabase = window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

**C. Replace the storage functions.** Find this whole block:

```js
function loadOptions(){
  try{
    const raw = localStorage.getItem(OPT_KEY);
    if(raw) return JSON.parse(raw);
  }catch(e){}
  localStorage.setItem(OPT_KEY, JSON.stringify(DEFAULT_OPTIONS));
  return JSON.parse(JSON.stringify(DEFAULT_OPTIONS));
}
function saveOptions(opts){ localStorage.setItem(OPT_KEY, JSON.stringify(opts)); }

function loadPresets(){
  try{
    const raw = localStorage.getItem(PRESET_KEY);
    if(raw) return JSON.parse(raw);
  }catch(e){}
  localStorage.setItem(PRESET_KEY, JSON.stringify(DEFAULT_PRESETS));
  return JSON.parse(JSON.stringify(DEFAULT_PRESETS));
}
function savePresets(p){ localStorage.setItem(PRESET_KEY, JSON.stringify(p)); }

let options = loadOptions();
let presets = loadPresets();
```

Delete it, and paste this in its place:

```js
async function loadOptions(){
  const { data } = await supabase.from('utm_builder_data').select('value').eq('key','options').maybeSingle();
  if(data && data.value) return data.value;
  await saveOptions(DEFAULT_OPTIONS);
  return JSON.parse(JSON.stringify(DEFAULT_OPTIONS));
}
async function saveOptions(opts){
  await supabase.from('utm_builder_data').upsert({ key:'options', value:opts });
}
async function loadPresets(){
  const { data } = await supabase.from('utm_builder_data').select('value').eq('key','presets').maybeSingle();
  if(data && data.value) return data.value;
  await savePresets(DEFAULT_PRESETS);
  return JSON.parse(JSON.stringify(DEFAULT_PRESETS));
}
async function savePresets(p){
  await supabase.from('utm_builder_data').upsert({ key:'presets', value:p });
}

let options = {};
let presets = [];
```

**D. Replace the init section at the very bottom of the script.** Find:

```js
// ---------- Init ----------
renderFieldSelects();
renderPresetSelect();
renderManageOptions();
renderPresetManageList();
```

Replace it with:

```js
// ---------- Init ----------
async function init(){
  options = await loadOptions();
  presets = await loadPresets();
  renderFieldSelects();
  renderPresetSelect();
  renderManageOptions();
  renderPresetManageList();
}
init();
```

Save the file.

### Step 5 — Every "save" call needs one small tweak

Anywhere the code calls `saveOptions(...)` or `savePresets(...)` and then immediately re-renders, add the word `await` in front of it, and mark the surrounding function `async`. There are four places — search the file for `saveOptions(options)` and `savePresets(presets)` and add `await` before each:

- Inside the "remove" button handler in `renderManageOptions()`
- Inside the "add" button handler (`doAdd`) in `renderManageOptions()`
- Inside the `savePresetBtn` click handler
- Inside the delete button handler in `renderPresetManageList()`

For each of those, add the word `async` right before `function` (or before `()=>{` for arrow functions) in that same handler, and put `await` in front of the save call. For example:

```js
// before
rm.addEventListener('click', ()=>{
  options[f.key] = options[f.key].filter(v=>v!==val);
  saveOptions(options);
  ...
});

// after
rm.addEventListener('click', async ()=>{
  options[f.key] = options[f.key].filter(v=>v!==val);
  await saveOptions(options);
  ...
});
```

### Step 6 — Test it

1. Open `utm-builder.html` in your browser.
2. Add a new option or preset.
3. Open the same file in a **different browser** (or an incognito window). If the value you added shows up there too, it's working — you're now reading from the shared database instead of your own browser.

### Step 7 — Share with your team

- If hosting on **GitHub Pages**: push the updated file to your repo, turn on Pages in repo settings, and send your team the Pages URL.
- Everyone who opens that same URL will now see and edit the same shared list of options and presets.

---

## Quick reference

| You want to... | Do this |
|---|---|
| Change starting dropdown values | Edit `DEFAULT_OPTIONS` (Part 1.1) |
| Change starting presets | Edit `DEFAULT_PRESETS` (Part 1.2) |
| Change colors or labels | Edit `:root{}` or `FIELDS` (Part 1.3) |
| Make options shared across your team | Add Supabase backend (Part 2) |
