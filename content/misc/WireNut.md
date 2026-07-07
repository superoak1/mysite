---
title: "WireNut Electrician Scraper"
date: 2026-05-12
tags: ["Fun Project", "HTML", "Python"]
draft: false
---

Check out [WireNut Electrician Scraper](https://github.com/superoak1/WireNut-Locator)

**WireNut Scraper: How I Automated My Lead Generation**

Have you ever worked a shitty sales job where you had to prospect your own leads? I have and it was the worst, so I built this tool to generate leads for me without having to put in much work.

**What is it?**

WireNut Scraper is a Windows desktop app that pulls licensed electrical contractor data from two official government registries — TSBC (Technical Safety BC) and ESA (Electrical Safety Authority Ontario) — and dumps the results straight into a Google Sheet. Enter a city, click Scrape, done.

No manual searching. No copy-pasting. No wasted afternoons.

**How it works**

The app runs a headless Chromium browser in the background using Playwright, a browser automation library. For TSBC it navigates the search page, filters by electrical contractors, and scrapes each results page. For ESA it hits a hidden data endpoint that powers their contractor map, pulling down the full dataset of 18,000+ licensed Ontario contractors in one shot, then filters by city.

Results get written directly to a Google Sheet using the Sheets API — organized by city, one tab per search.

The whole thing is packaged as a standalone Windows EXE using PyInstaller with Chromium bundled alongside it, so whoever uses it doesn't need Python or any other setup.

**The outcome**

What used to take an hour of manual searching per city now takes about 30 seconds. Search Lindsay, Ontario? 27 licensed electrical contractors with names, phone numbers, addresses, and licence profile links — ready to hand off or work through.

The tool is open source on GitHub if you want to check it out or adapt it for your own use case.