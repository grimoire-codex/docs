---
layout: home

hero:
  name: Grimoire
  text: Self-Hosted TTRPG Library Manager
  tagline: Browse, search, and read your entire tabletop RPG PDF collection from any device.
  image:
    src: /logo.png
    alt: Grimoire
  actions:
    - theme: brand
      text: Get Started
      link: /guide/getting-started
    - theme: alt
      text: Compose Generator
      link: /compose-generator
    - theme: alt
      text: GitHub
      link: https://github.com/hunter-read/grimoire

features:
  - icon: 📚
    title: Library Browser
    details: Organizes your collection by game system with automatic folder detection and category mapping.
  - icon: 🔍
    title: Full-Text Search
    details: Every page of every PDF is indexed with SQLite FTS5 for instant, accurate search across your entire library.
  - icon: 📖
    title: Page-by-Page Viewer
    details: PDFs rendered as images for fast mobile viewing with pinch-to-zoom, swipe navigation, and spread mode.
  - icon: 🗺️
    title: Map Gallery
    details: Browse battlemaps by directory with tag filtering, grid metadata, and full-resolution download.
  - icon: 🔊
    title: Audio Library
    details: Browse ambient tracks, music, and sound effects by folder with a persistent global player that keeps playing as you navigate.
  - icon: 🎭
    title: Campaigns
    details: Track GM-run and personal campaigns with a markdown notes wiki, character sheets, linked resources, guest invites, and scheduling.
  - icon: 📡
    title: OPDS Catalog
    details: Each user can generate a personal OPDS feed URL to connect e-reader apps directly to their library.
  - icon: 🔐
    title: OpenID Connect
    details: "Delegate authentication to any OIDC-compliant identity provider: Authentik, Keycloak, Auth0, and more."
  - icon: 🐳
    title: Docker Ready
    details: One command to run. Mount your library directory, set a secret key, done.
---

::: tip Looking for a group to play with?
I also built [findtabletop.games](https://findtabletop.games), a free way to find tabletop RPG groups and players. Grimoire handles your library; findtabletop.games helps you find the table.
:::
