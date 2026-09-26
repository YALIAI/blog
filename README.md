# Autonomous Driving Notes

Yali Nie's autonomous driving learning blog. Static HTML, CSS, and JavaScript, hosted directly with GitHub Pages. No build tools required.

## Preview locally

```bash
python3 -m http.server 8000
```

Open `http://localhost:8000/`.

## Update content

- Add an HTML article in the repository root, then link to it from the “Recent notes” section of `index.html`.
- Update the “Technology news” links in `index.html`.
- Add technical resources to the `resources` array in `site.js`. Supported categories are `datasets`, `perception`, `simulation`, and `courses`.
- Edit `style.css` for visual changes.

## Publish

GitHub Pages deploys from the `main` branch at the repository root. Commits to `main` trigger a new deployment.

The notes and descriptions here are personal learning material. Refer to the linked projects for current configuration and dataset licenses.
