# Community Projects

A community-curated list of third-party tools built around Grimoire.

::: warning These are not maintained by Grimoire
Everything on this page is written and maintained by the community, not by the
Grimoire project. Nothing here has been audited or endorsed, and a tool that
writes to your library or your database can damage it. Read the source, check
what permissions it asks for, and keep a [backup](/configuration/backups)
before pointing anything new at a library you care about.
:::

Building something? Open a pull request against the
[docs repo](https://github.com/hunter-read/grimoire) to add it here. Please keep
the list to tools that are working and usable today rather than early
experiments, and add your entry in alphabetical order.

## grimoire-cli

[thomaslazar/grimoire-cli](https://github.com/thomaslazar/grimoire-cli) — a
command-line client for Grimoire covering metadata management, system and book
lookups, library scans, backups, and add-on management. Output is always JSON on
stdout with logs and errors on stderr, which makes it easy to pipe into scripts
or hand to an agent. Ships as a self-contained Native AOT binary (.NET 10, no
runtime to install) via Homebrew, install scripts, Debian packages, or a direct
download. MIT licensed.
