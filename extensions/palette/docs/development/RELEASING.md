# Release checklist

## Before tagging

- [ ] Set the correct public repository metadata in `spindle.json` if needed.
- [ ] `package.json` and `spindle.json` versions match the intended release tag.
- [ ] `GUIDE.md` and `src/ui/guide.ts` identify the intended public version.
- [ ] Run `bun run verify` locally.
- [ ] Confirm rebuilt `dist/frontend.js` and `dist/backend.js` are committed.
- [ ] Fresh-install from the GitHub repository into a clean Lumiverse extension slot.
- [ ] Existing-project migration smoke: load a schema-v39 project and confirm projects/My Styles survive.
- [ ] Picker: persistent pick, Edit Part, Browse Inside, Assistant/User/Both.
- [ ] Groups: retarget member, Add sibling, create a valid shared-parent group.
- [ ] Read Style / Read Page: inspect without eager CSS ownership.
- [ ] Boost: Colors, Backdrop, Typography-only, continuity during commits.
- [ ] My Styles: Target, Component, Bundle; cross-project apply and reload.
- [ ] Packs: apply/reset at least Manga + Visual Novel on desktop/mobile.
- [ ] Media: raw prose image + native attachment, Auto source, Media Flow.
- [ ] Float/mobile: top/bottom edge, resize, hide/show mini widget.
- [ ] Guide jump links scroll correctly inside the Lumiverse Guide modal.
- [ ] Verify license, unofficial-project statement, and third-party attribution remain present.

## GitHub release

Suggested tag: `v1.0.0`

Attach the runtime ZIP produced for the release and paste the prepared release notes. Keep the complete human-readable source public in the repository.
