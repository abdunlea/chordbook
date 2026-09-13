# Gig Chordbook

A single-page chord/lyric library for a cover band: one song per page, chords in a boxed grid, a drag-to-reorder gig setlist, and a print-ready PDF export. Runs entirely in the browser — no server, no build step.

## PIN gate

The page asks for a PIN before showing the songbook (current PIN: `asr`). This is **not real security** — it's plain browser JavaScript with no server to check a password against, so anyone willing to look at the page's source could work around it. It's meant only to keep a casual visitor who stumbles on the link from poking around. Once you enter it correctly on a device, that browser remembers you're unlocked (via local storage) so you won't be asked again there.

To change the PIN, open this page, open your browser's developer console, and run:

```js
crypto.subtle.digest('SHA-256', new TextEncoder().encode('yournewpin')).then(buf =>
  console.log(Array.from(new Uint8Array(buf)).map(b=>b.toString(16).padStart(2,'0')).join(''))
)
```

Copy the hash it prints, then open `index.html`, find the line starting with `const PIN_HASH =`, and replace the long hex string with your new one. Commit the updated file.

## How the data works

- `songs.json` is the song database. It's a plain JSON array sitting next to `index.html` in this repo.
- When the page loads, it reads your library from this browser's local storage if there's a working copy, otherwise it loads `songs.json` fresh.
- Every change you make (adding a song, editing one, importing a CSV) is saved automatically **to this browser only**. The status pill at the top tells you whether the browser copy still matches `songs.json` on disk.
- To make changes permanent — and visible from any other browser or device — click **Download songs.json**, then replace the file in this repo with the one you downloaded (drag-and-drop upload on github.com's file page, or copy it over your local clone and `git push`).
- **Reload songs.json** does the opposite: it throws away the browser-only copy and pulls the committed file back in. Use it after you've edited `songs.json` by hand, or to discard changes you don't want to keep.

This is a deliberate, static-site-friendly design: there's no backend, so nothing can silently rewrite files in your repo. Saving to GitHub is always something you choose to do, by downloading and committing the file yourself.

Saved gig lists work the same way, in a second file, `setlists.json`:

- **Save** updates the gig list currently loaded in the "Load saved gig…" dropdown (or prompts for a name if you haven't loaded one — e.g. building a brand-new list from scratch).
- **Save as new list…** always creates a separate saved list under a new name, even if one is already loaded. This is the one to use when you load an old setlist as a starting point for a new gig — pick it from the dropdown, reorder/add/remove songs, then Save as new list so the original stays untouched and your edits land in a new entry.
- **Download setlists.json** / **Reload setlists.json** work exactly like their `songs.json` counterparts — download and commit to make your saved gig lists permanent and visible on other devices; reload to discard browser-only changes and pull the committed file back in.

The setlist you're actively building (not yet saved under a name) stays in browser local storage only, same as before — there's no need to commit a file every time you're just trying out an order.

## Finding a song in a big library

Once your library grows, the row of letters above the search box (an A–Z jump bar) lets you jump straight to songs starting with a given letter instead of scrolling — greyed-out letters mean there's nothing there yet (either you have no songs starting with that letter, or your current search has filtered them out).

## Setting it up on GitHub Pages

1. Create a new GitHub repository (public or private — private repos can still use Pages on most plans).
2. Add `index.html`, `songs.json`, and `setlists.json` (from this folder) to the repo, at the top level.
3. In the repo, go to **Settings → Pages**. Under "Build and deployment", set Source to "Deploy from a branch", pick your default branch (usually `main`) and the `/ (root)` folder, then save.
4. GitHub will give you a URL like `https://<your-username>.github.io/<repo-name>/` — that's your chordbook, live in a minute or two.
5. Bookmark it, or add it to your phone's home screen for easy access at a gig.

## Bulk-adding songs from a spreadsheet

Use **Export CSV** to download your current library as a CSV — open it in Google Sheets and it doubles as a template showing the exact column layout: `Title, Section, Chords, Lyrics, Notes, Layout, Text Size %, Chord Size %`.

- Give a song's chord grid one line per chord row inside the "Chords" cell (in Sheets, press Alt+Enter for a new line within a cell), with chords separated by commas — e.g. one line `C, Am, F, C`, the next `F, G, C, C`.
- A song with more than one section (Verse, Chorus, etc.) gets an extra row right after the first — leave Title blank on that row, and just fill in Section and Chords.
- `Layout` accepts `auto`, `1`, `2`, or `side`; `Text Size %` and `Chord Size %` are whole-number percentages (100 = normal size).

Save your sheet as CSV and use **Import CSV** to bring it in. Import never overwrites an existing song — if an imported title matches one you already have, the new one is added with "(copy)" on the end so you can find it later (search "copy" in the library) and decide what to keep. As with any other change, remember to **Download songs.json** afterward to make the import permanent.

## Notes

- PDF export uses [jsPDF](https://github.com/parallax/jsPDF) and [html2canvas](https://github.com/niklasvh/html2canvas), loaded from cdnjs. CSV parsing uses [PapaParse](https://www.papaparse.com/), also from cdnjs. All three are loaded via `<script>` tags in `index.html` — no npm install needed.
