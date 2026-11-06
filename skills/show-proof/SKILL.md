---
name: show-proof
description: Use when you finish a task that changes something a user can see, such as UI, a page, a game, or a rendered document, and before saying it is done. Also use when the user asks to see the result.
---

# Show Proof

When a task changes what someone sees, show the result. Take a screenshot of
the real thing, look at it yourself, and give it to the user with your final
message.

Skip this for changes with nothing to see: backend logic, refactors, config,
docs. Test or command output is the proof there. Do not take a screenshot just
to have one.

## Capture

1. Run the real app or open the real file. Reuse a running dev server if there
   is one. Stop anything you started.
2. Get to the state the task changed, with realistic data. A page that only
   shows a spinner or an empty state proves nothing.
3. Capture it. In T3, use a headless browser such as Playwright or Puppeteer.
   Do not use `preview_*` for this unless the user asked for browser work. For
   a simulator or emulator, use `device_list`, `device_open`, and
   `device_screenshot`, not `xcrun simctl`. Outside T3, use the harness's
   browser tools, Playwright, Puppeteer, or
   `xcrun simctl io booted screenshot <path>`.
4. Capture each state the task touched, and only those. For a bug fix, show
   before and after if you can. If the task touched a responsive layout, add a
   mobile width.

If a project file says to ask before browser automation, running the app, or
screenshots, ask first. `full-send` and `full-stop` already grant this.

Save images outside the tracked tree, in a temp directory or a gitignored
folder. Never commit them unless the user asks.

## Look at it

Open each image and read it. Check that it shows the change, and look for
broken layout, cut-off text, missing images, and console errors. If something
is wrong, fix it and capture again. Do not hand over an image you have not
looked at.

## Show it

In T3, show the images in the thread. A path alone does not show them:

1. Write a small self-contained HTML page with one `<img>` per image, using its
   absolute path, and a short caption. T3 inlines local images.
2. Pass it to `html_preview`. Fix anything in `missingImages`.
3. Pass it to `html_render` with the `contentHeight` from `html_preview`. Do not
   start a server for it.

Outside T3, show the images if the harness can display them.

In your final message, give each image's absolute path and one line on what it
shows.

If you could not capture it, say so and say why. Never say a visual change
works when you have not seen it.

For a PR, attach the images only if you have a way to upload them. First check
that they show no secrets, personal data, or real customer data.
