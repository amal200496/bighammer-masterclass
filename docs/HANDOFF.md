# Handoff: go-live build (co-host refresh)

This commit is the go-live version of the masterclass page. One file, `index.html`, fully self-contained (images are embedded).

## What changed in this commit
- New host portraits for Srinath Reddy and Richard Lawrence. Both are black and white on white, 4:5, 800x1000, with the same face size and eye line, so they read as a matched pair.
  - Srinath: re-cut from the original BigHammer studio photo (higher resolution than the old crop).
  - Richard: from his Kie.ai avatar set (half-body studio pose), background removed and toned to match.
- The images live in two CSS variables, `--photo-s` and `--photo-r`. They feed both places portraits appear: the hero host strip and the "Your hosts" section.
- Both people are now presented as co-hosts: "Host" label on both, "Your hosts" heading, the "Joined by" tag removed, same size and position for both cards, designations shown under each name.
- Hero lead line and social description now name both hosts.

## Second pass (polish and content)
- Richard's portrait replaced with a new Kie.ai pose from his avatar set: light closed-mouth smile, black blazer and tee on white, matching Srinath's photo.
- Both host cards now have three pointers under the designation (Richard's quote removed so the cards match). Richard's pointers are role-level; confirm the wording with him.
- "What you get" rebuilt from the masterclass deck: four modules with detail (business problems, six waste patterns, healthcare teardown numbers, assess/migrate/monitor), the live Q&A, and the two ways to run the free assessment.
- Free bonus under the form is a highlighted card; the form testimonial now has Anushka's photo (from bighammer.ai) in a small circle.
- Removed: footer (privacy and terms links), the recording promise under the button and in the final CTA, the scan line that crossed the workload labels.
- Fixes found in the visual pass: headline no longer shows "0%" while counting, proof numbers share one size, mobile proof strip no longer overflows, "Re-assess" label no longer clipped, pattern 05 graphic ends on the tiny-files grid, host cards line up.

## Third pass (registration flow and thank-you page)
- The form now asks for full name, work email and phone number (country picker, validated, stored in international format such as +919876543210). The old optional second step (company, role, phone) is gone.
- On submit the page runs the same flow as the current live page (webinar.bighammerai.com): WebinarGeek registration through `broadcast.growthclub.org`, the n8n webhook, and GoHighLevel external form tracking (which drives the WhatsApp message). Then it redirects to `thank-you.html`.
- `thank-you.html` is new. It carries the same four steps as the live thank-you page (join link, WhatsApp, book a demo, email confirmation) plus add-to-calendar buttons, in the new design.
- Safety switch: the real calls only run when the page is served from a host listed in `LIVE_HOSTS` (default `webinar.bighammerai.com`). Anywhere else the sign-up is simulated and the thank-you page shows a yellow "Preview mode" bar.

## Fourth pass (integration parity with the live page)
- The form id is `registrationForm` again, the same as the live page. GoHighLevel's tracker names each submission by the form's id, so a different id would arrive in GHL as a new form and the existing workflow (tags, WhatsApp) would not start. Do not rename it.
- `workers/automations/` is copied from the old project: the Cloudflare Worker that turns WebinarGeek webhooks into GHL tags and custom fields, plus the beehiiv nurture routes. It is one deployed Worker and does not depend on which landing page is live. Secrets (`.dev.vars`) were not copied.
- Checked with a local dry run (outbound calls recorded, not sent): one GHL form submission with `full_name`, `email`, `first_name`, `last_name`, `phone`, `phone_number`, `readable_webinar_date`, `readable_webinar_time`, `watch_link_webinargeek`; the n8n payload with UTM fields; the WebinarGeek registration; the thank-you page with join link and WhatsApp link.
- Not the same as the live page: company and role are not collected, and the date shown on the page is fixed text (the live page read it from the broadcast backend).

## Go-live steps
1. Upload `index.html` and `thank-you.html` to the root of webinar.bighammerai.com (replacing the current two files). Keep the file name `thank-you.html`; the form redirects to it.
2. If the page will live on any other domain, add that hostname to `LIVE_HOSTS` near the top of the registration script in `index.html` (search for `LIVE_HOSTS`).
3. Confirm the IDs at the top of the registration script still match the current live page: `BROADCAST_ID` (6841508), `CLIENT_ID`, `N8N_WEBHOOK_URL`, `GHL_TRACKING_ID`. On thank-you.html, `WHATSAPP_NUMBER`.
4. Test with a real sign-up on the live domain and check: the contact lands in GHL with phone, first and last name; the n8n row appears; the WebinarGeek registration exists; the WhatsApp message arrives; the thank-you page shows the "Access webinar seat" button with the personal join link.
5. Company and role are no longer collected. The n8n payload sends them as empty strings, so check nothing downstream requires them.
6. Optional: add `og:url`, `og:image` and a `canonical` link once the final URL is known.
