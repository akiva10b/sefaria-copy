# Copy Tool

This lightweight HTML page powers the “Copy Text” plugin UI. It lets users pull Hebrew/English text from Sefaria and drop it directly into the clipboard with minimal formatting.

## Features
- Requests current selections (`sref`) from the host application via `postMessage`.
- Fetches text from Sefaria’s endpoint `https://www.sefaria.org/api/texts/{ref}` with `context=0`, `stripItags=1`, and `fallbackOnDefaultVersion=1`.
- Normalizes the API response, stripping any HTML from English (`text`), Hebrew (`he`), and reference fields (`ref`, `heRef`).
- Copies any mix of:
  - Segment vs. entire section references.
  - English translation and/or Hebrew source.
  - English reference label and optional Hebrew reference label (`heRef`).

## Workflow
1. Click **Request sref** to ask the host for the active reference (the host should reply with `type: "sref:response"`).
2. Choose the scope (segment vs. section) and which fields to include.
3. Press **Copy**. The tool:
   - Calls the Sefaria API for the target ref.
   - Normalizes the response and strips markup.
   - Builds the clipboard text in the order: English ref → Hebrew ref → body blocks (if enabled).
   - Uses the Clipboard API (falling back to `document.execCommand`) to place the text on the clipboard.
4. Status messages show success or a readable error.

## Developer Notes
- `flattenText()` collapses nested arrays/objects returned by Sefaria into newline-delimited strings.
- `stripHtml()` relies on a shared in-memory `<div>` to remove tags without external dependencies.
- Copy is blocked if the user deselects every content option (ref, heRef, translation, source).
- The host page must listen for `type: "plugin:ready"` and `type: "plugin:request-sref"` to complete the messaging loop.

## Testing Tips
- Try a single verse (`Esther 1:1`) and a chapter (`Esther 1`) to confirm both string and array responses work.
- Toggle each checkbox combination to verify the clipboard output order and the “select something to copy” validation.
- Check the browser console for network errors; the tool surfaces HTTP and API-level errors in the status bar.
