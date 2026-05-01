# happy-forms-copy

Tiny static page hosted on GitHub Pages that handles the **Copy button click flow** for [`happy-forms`](https://github.com/RHSTheSecond/happy-forms) (and `happy-fb-forms`).

## Why this exists

Apps Script's `HtmlService` forces all served HTML into a sandboxed iframe at `script.googleusercontent.com` with a Permissions Policy that blocks `navigator.clipboard.writeText()` and `document.execCommand('copy')`. Result: the auto-copy-and-close flow we built into the cardsV2 Copy button never closes the tab silently. Tab stays open with a visible manual-copy UI.

Hosting the copy page **outside** Apps Script (here, on GitHub Pages, served at the top level — no iframe) bypasses that restriction. Clipboard API works normally; `window.close()` succeeds; the tab opens for ~150ms and closes itself before the user perceives more than a flash.

## How it works

1. Chat user clicks a `📋 COPY` button on a card row
2. Button is `openLink` to `https://rhsthesecond.github.io/happy-forms-copy/?value=<url-encoded value>&label=<url-encoded label>`
3. New tab opens, this page loads
4. JS calls `navigator.clipboard.writeText(value)` → succeeds (top-level context, recent user activation from the Chat click)
5. JS calls `window.close()` → tab closes before user fully sees it
6. User stays on Chat with the value in their clipboard

If the modern Clipboard API fails for any reason, fallback path tries legacy `document.execCommand('copy')` with a hidden textarea. If that also fails, the page renders the value selected with a "Press Cmd+C" hint.

## Privacy / security

- The page contains **zero secrets**. It just reads URL parameters and writes them to clipboard.
- The `value` is in the URL query string, so it's logged in browser history and any reverse-proxy logs. Don't pass anything truly sensitive.
- Public repo because GitHub Pages on Free/Pro plans serves public repos cleanly. No reason to make this private — there's nothing here that leaking would matter.

## Updating

Edit `index.html` → `git push`. GitHub Pages re-deploys automatically (~30s).
