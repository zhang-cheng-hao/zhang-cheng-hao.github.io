# Chenghao Zhang Academic Homepage

This is a customized academic homepage built from
[luost26/academic-homepage](https://github.com/luost26/academic-homepage).

## GitHub Pages

Create a GitHub repository named:

```text
zhang-cheng-hao.github.io
```

Then push this directory to that repository. The site is configured for a user
site, so `_config.yml` uses:

```yaml
baseurl: ""
url: "https://zhang-cheng-hao.github.io"
```

In GitHub, open **Settings -> Pages** and deploy from the `main` branch.

## Local Preview

Install Ruby/Jekyll dependencies once:

```bash
bundle install
```

Then run:

```bash
bundle exec jekyll serve
```

Open the local URL printed by Jekyll.

## Content Files

- `_data/profile.yml`: name, email, GitHub, education, honors, CV link.
- `_publications/`: publication entries.
- `_news/`: homepage news entries.
- `index.html`: English homepage.
- `zh.html`: Chinese homepage.
- `docs/`: curated public notes distilled from local preparation material.

## Curated Notes

- `docs/interview-prep-handbook.md`: structured interview-preparation handbook distilled from historical local notes and chat-generated material.
- `docs/source-material-map.md`: map of local source material and what is intentionally not committed.

## Template Credit

The site keeps attribution to the original `academic-homepage` template in the
footer.
