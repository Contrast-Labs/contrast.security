
# contrast.security

## Using this site:

1. [Install Hugo](https://gohugo.io/overview/installing/)
2. Clone this repository
```bash
git clone https://github.com/Contrast-Labs/contrast.security
cd contrast.security
```
3. Run Hugo by using
```bash
hugo server
```

## Building this site:

1. Build static Hugo site by running:
```bash
hugo
```
2. Get ready for GitHub Pages deployment by changing `public/` to `docs/` by running:
```bash
rm -rf docs && mv public docs
```
3. Push to GitHub
```bash
git add .
git commit -m 'commit message'
git push -u origin main
```

## Adding articles

Adding blog posts, etc is as simple as adding a [new markdown file here](https://github.com/Contrast-Labs/contrast.security/tree/main/content/post).

You will then need to rebuild the site with hugo and push it up to this repo. A Github pages bot will auto update contrast.security momentarily afterwards.

Things to keep in mind for adding a new article.  

1. Make sure to follow the standard heading (example below).
2. Make sure the author exists that you reference.
3. Pay attention to date format.
4. Use the feature flag if you would like to focus on it.
5. Try to reuse previous tags.
6. Lastly, make sure you copy the thumbnail image out [here](https://github.com/Contrast-Labs/contrast.security/tree/main/static/images).  Please note, there is a night/day theme option, so make sure your image works in both night and day.

```javascript
+++
author = ["matt-austin"]
title = "Contrast Labs Reveals Dependency Confusion Vulnerability in Microsoft Teams"
date = "2021-03-04"
description = "Matt Austin reviews the risk of dependency confusion."
tags = [
    "hacked",
    "dependency-confusion",
]
featured = true
thumbnail = "images/microsoft-teams.png"
+++
```

## Adding authors 

Adding authors is a `json` file that goes [here](https://github.com/Contrast-Labs/contrast.security/tree/main/data/authors).  Be sure to add a photo [here](https://github.com/Contrast-Labs/contrast.security/tree/main/static/images/authors) if you'd like to.  Please add yourself upon publishing your first article.

You will then need to rebuild the site with hugo and push it up to this repo. A Github pages bot will auto update contrast.security momentarily afterwards.
