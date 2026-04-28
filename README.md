# Paulo H. Barchi - Personal Blog & Technical Writings

This repository contains my personal blog and technical writings, built with **Hugo** and the **PaperMod** theme.

## Transition from Academic Site
This blog site replaces my previous academic website - current repository for it: https://github.com/paulobarchi/academic-archive.

## Technical Stack
- **Framework:** [Hugo](https://gohugo.io/)
- **Theme:** [PaperMod](https://github.com/adityatelange/hugo-PaperMod)
- **Deployment:** GitHub Pages (Automated via GitHub Actions)

## Interactive Features
I've integrated several lightweight, developer-centric features to foster engagement without sacrificing performance:

- **Comments:** Powered by [Giscus](https://giscus.app/), utilizing GitHub Discussions for a seamless, markdown-supported comment system.
- **Reactions:** A Twitter-style "Like" system provided by [Lyket](https://lyket.dev/).
- **Visit Counter:** Non-intrusive page view tracking via [Busuanzi](http://busuanzi.ibruce.info/).

## Local Development
To run this blog locally, ensure you have Hugo installed:

```bash
# Clone the repository with submodules
git clone --recursive https://github.com/paulobarchi/paulobarchi.github.io.git

# Start the Hugo server
hugo server -D
```

The site will be available at `http://localhost:1313`.
