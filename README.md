# Clearpath ADHD Workspace

A mobile-friendly, low-friction personal task and focus system, replacing an old Cloudinary demo repository.

## Hosted version

Static site designed for GitHub Pages, using the repository root as publishing source.

### Features

- Capture tasks quickly (Ctrl/Cmd + K).
- Today, Inbox, Tasks, Projects, Notes and Focus views.
- Top three tasks, priority flags and overdue indicators.
- Assign tasks to projects, record notes, mark tasks done.
- 25-minute focus timer and completed-session log.
- Import and export JSON backups.

### Data and privacy

**This release is browser-local.** Records are stored in `localStorage` on the current browser/device, not on GitHub or the server. A browser reset or changing devices can lose access to data. Use **Export backup** regularly. No account sign-in, cloud sync, MCP endpoint, or AI assistant is enabled in this hosted release. Do not rely on it for confidential client work or as a sole archive.

### GitHub Pages deployment

In **Settings > Pages**, set **Deploy from a branch**, choose `master` and `/(root)`. The site will be hosted at `https://jibrilmohamod.github.io/cloudinaryDemo/`.

Once GitHub Pages is enabled, changes pushed to `master` publish automatically. The former demo is preserved on branch `archive/cloudinary-demo-2023`.

### Next milestone

Migrate to Supabase Auth/Postgres with RLS, add an authenticated OpenAI API assistant and per-user OAuth-protected MCP server, then migrate selected Notion databases and add email/calendar integration. Those are not enabled in this static preview.
