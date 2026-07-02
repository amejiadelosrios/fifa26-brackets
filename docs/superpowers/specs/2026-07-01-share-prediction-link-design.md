# Share Prediction via Link — Design

## Purpose

Let someone name their bracket prediction and share it with others (friends/family
group) so they can view that specific prediction, without adding any backend or
database to the project.

## Constraints

- No database, no backend service. The app stays a static site on GitHub Pages.
- No central gallery/list of everyone's predictions inside the app — sharing is
  peer-to-peer (each person sends their own link, e.g. in a group chat).
- Must keep working as a single-file, no-build app (`index.html`).

## Data encoding

A shared prediction is the pair `{ name, winners }` (same shape as the app's
existing `state.winners` map), encoded into a URL query parameter:

1. `JSON.stringify({ n: name, w: winners })`
2. UTF-8 safe base64 encode (so accented names survive): `btoa(unescape(encodeURIComponent(json)))`
3. Make it URL-safe: replace `+` → `-`, `/` → `_`, strip trailing `=` padding

Result goes in the URL as `?p=<encoded>`, e.g.
`https://amejiadelosrios.github.io/fifa26-brackets/?p=eyJuIjoiSnVhbiIs...`.
Decoding reverses each step. `winners` has at most 17 entries (R16 8 + QF 4 + SF 2
+ 3RD 1 + FINAL 1 + CHAMPION 1), so the encoded string stays a few hundred
characters — well within URL limits for chat apps.

## UI changes

### Name field + share button (interactive mode)

- A text input "Tu nombre" near the header, persisted to `localStorage`
  (`fifa26-bracket-name`) so it's remembered across visits.
- A "Compartir mi predicción" button. On click:
  - If the name field is empty, focus it and stop (require a name before sharing).
  - Build the share URL from the current `state.winners` and the name.
  - If `navigator.share` is available (mobile), open the native share sheet with
    that URL.
  - Otherwise, copy the URL to the clipboard (`navigator.clipboard.writeText`) and
    show a brief inline confirmation ("Link copiado").

### Read-only viewing mode

Triggered when the page loads with a `?p=` query parameter present:

- Decode `{ name, winners }` from the parameter. If decoding fails (malformed
  link), fall back to normal interactive mode and ignore the parameter.
- Skip `loadPersisted()` — the viewer's own saved local predictions are not
  touched or mixed in.
- Render the bracket using the decoded `winners` instead of local state.
- Disable dragging entirely: `renderSlot` does not attach `pointerdown` handlers
  or the `draggable` class when in read-only mode.
- Hide the "Reiniciar predicciones" button, the name field, and the share button
  (none apply to someone else's read-only bracket).
- Show a banner at the top: "Viendo la predicción de **{name}**" with a button
  "Volver a mi predicción" that navigates to the bare URL (`location.pathname`,
  no query string), reloading the viewer's own interactive bracket.

### Mode selection

A single `READ_ONLY` boolean, computed once at startup from
`new URLSearchParams(location.search).get('p')`, gates: which state populates the
bracket (decoded vs `loadPersisted()`), whether `renderSlot` attaches drag
handlers, and which header controls are shown.

## Out of scope

- Any central listing/gallery of submitted predictions.
- Editing or "adopting" someone else's shared bracket as your own.
- Any server-side validation of shared data (a malformed/tampered `?p=` link
  either fails to decode — falls back to interactive mode — or decodes into
  whatever plausible-looking bracket it encodes; there's no way to verify
  authenticity without a backend, which is an accepted trade-off).
