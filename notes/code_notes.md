---
title: How to generate and maintain this site
tags:
  - gatsby
  - code-notes
emoji:
---

## Introduction
As the name suggested, this site helps me remember all the code-related knowledge that I need. Usually it is difficult to remember how to update and maintain sites like this. That is the main purpose of this entry.

The whole system consists of three parts
- Contents, which is a list of markdown files with a specific header.
- Webpage generation is done by a [gatsby plugin](https://www.gatsbyjs.com/plugins/gatsby-theme-code-notes/)
- [gh-pages](https://www.npmjs.com/package/gh-pages) publishes the webpage using GitHub Page.

## How to make the system
1. Install gatsby-cli. I did `npm install -g gatsby-cli`
2. Start a new code-note project by `gatsby new code-notes https://github.com/MrMartineau/gatsby-starter-code-notes`
3. The project is already a git repository. Link that to a GitHub repository. I named it `code-notes`.
4. Follow the official instruction to generate the website and publish to GitHub Page. I specifically did
  - **Within the repository**, install gh-pages. I did `npm install gh-pages --save-dev`
  - Added `pathPrefix: "/code-notes",`(the repository name) to `module.exports` in the file `gatsby-config.js`
  - Added `"deploy": "gatsby build --prefix-paths && gh-pages -d public"` to `"scripts"` in `package.json`
  - Created a branch called `gh-pages`
  - `npm run deploy` will generate the website and check-in codes to the `gh-pages` branch automatically.
5. In the GitHub webpage, go to the settings of the repository to let GitHub Pages to host the project page. Choose `gh-pages` branch and `/(root)` as the path. The code-notes webpage shows up in `<username>.github.io/code-notes.`

## How to write a new entry
1. Add a markdown file under the `notes` directory. About the format, check an [example](https://github.com/mrmartineau/gatsby-starter-code-notes/blob/master/notes/example-note.md) or just copy from an existing one.
2. `npm run deploy` to generate and publish.
3. Version control the notes as your wish.

## References

- [Author's blog](https://zander.wtf/blog/code-notes-release)
- [GitHub](https://github.com/MrMartineau/gatsby-starter-code-notes)
- [How Gatsby works with GitHub Pages](https://www.gatsbyjs.com/docs/how-gatsby-works-with-github-pages/)