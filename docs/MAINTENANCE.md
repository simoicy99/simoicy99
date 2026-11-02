# Maintaining this profile

This repository is maintained with Claude-assisted automation. Notes for whoever edits it next:

- **Editorial preference.** This is a personal profile first, with the technical projects prominent. The introduction carries one subtle co-founder mention with a link to Restart, and nothing more about the company: no positioning, thesis, product description, or calls to action. The title is co-founder only, never CEO. The mastheads carry the name, the line "Systems, agents, and experiments.", the handle, and the location; no company name, title, or product terms appear in their visible or accessible text.
- **Claims.** Before adding any company fact, metric, or customer claim, confirm it with the founder and a public source. Describe linked projects by what their code does; do not assert who built them, when, or for whom unless that is documented.
- **Masthead.** `assets/identity-light.svg` and `assets/identity-dark.svg` share one layout. Edit the light file, then regenerate the dark one by swapping the palette: `#F3F0E8` to `#12120F`, `#14130F` to `#ECE8DD`, `#55524A` to `#A29D90`, `#A7B82A` to `#C2D040`. Keep them free of scripts, external fonts, and remote references. The README text must stay complete without the image.
- **Links.** README links use `blob/main` paths. If a file moves in a project repo, update the link. `docs/engineering-map.md` pins line links to specific commits; refresh the commit and line numbers when the code they describe changes.
- **Copy.** Keep the README short, in simple vertical sections that read well on a phone. No em dashes.
