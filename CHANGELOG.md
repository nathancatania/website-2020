# Changelog

All notable changes to this project will be documented in this file.

**This repository is now archived.**

## 2024-02-11
- Archived this version/repo of the website. Moved to CF pages + Mkdocs Material.
- Added demo/preview folder containing a video and screenshots of when this version of the site was live.
- Added v0.122 folder containing the changes I made to get the site working with Hugo v0.122 (from v0.108). These were never published in the end.

## 2022-12-09
- New post: Insecure Kubernetes for Evaluation Purposes

## 2022-12-09

- Updated post: Deploy Netskope Cloud Exchange
  - Updated the post to work with Cloud Exchange v4 as there were several installation changes from v3.
- Updated build to use latest Hugo version

## 2022-10-12

- New post: Netskope Quick Start Guide
- Added the ability to use callouts/admonitions (eg: warning!, heads up!, caution!) using a custom Hugo shortcode
  - `{{< callout success "Hint!" >}}This is a callout!{{< /callout >}}
  - This also leverages Bootstrap's [icon library](https://icons.getbootstrap.com/)
- Updated contact info
- Turned on global emoji usage in `config.toml`

## 2022-07-31
- New post: Deploy Apache Guacamole with SSL & SAML (Azure AD & Okta) integration

## 2022-03-06
- New post: How-to Configure SSO for Netskope Cloud Exchange

## 2022-03-01
- Blew away some dust.
- New post: Deploy Netskope Cloud Exchange

## 2020-10-06

- New Post: Deploy Zscaler Client Connector with Intune (iOS & Android)
- New Post: Deploy Zscaler Client Connector with Intune (Windows & macOS)

## 2020-09-06

- New Post: Configure Pi-Hole DNS + Cloudflare DNS over HTTPS (DoH) on a Raspberry Pi
  - This was a post I drafted in Feb, but never published for some reason.
- Fixed some text in my previous Sentinel & MCAS posts.

## 2020-08-09

- New Post: Integrate Zscaler with Azure Sentinel
- Fixed some typos on previous posts

## 2020-06-30
- New Post: Deploy Zscaler NSS in Azure
- New Post: Integrate Zscaler with Microsoft Cloud App Security (MCAS)
- Modified README to include running Hugo in dev environment with future-dated posts (-F) enabled
- Updated post: "Deploying ZPA ZEN Connectors" with notes regarding correctly pasting the API key

## 2020-05-17
- Moved website to Hugo + Netlify (from Jekyll + GitHub Pages - old repo [here](https://github.com/nathancatania/nathancatania.com))
- Modified [hello-friend-ng](https://github.com/rhazdon/hugo-theme-hello-friend-ng) theme as per the following:
  - Light/Dark theme backgrounds changed to #FFF and #000 (`variables.scss`)
  - Changed font to [Inter](https://rsms.me/inter/) (was Inter UI) (`_fonts.scss`, `_main.scss`)
  - Using 700 instead of 800 font-weight for bold/headings (`_fonts.scss`)
  - Added additional margin between h1/h2 headings (`_main.scss`)
  - Changed body letter-spacing to -0.011em (`_main.scss`)
  - Added border-radius to images (`_single.scss`)
  - Added top/bottom margin to images & figures in posts (`_single.scss`)
  - Centered all images & figures (`_main.scss`)
  - Tables were unstyled so copied style across from [original hello-friend theme](https://github.com/panr/hugo-theme-hello-friend) (`_main.scss`)
  - Removed time and timezone from post dates (`single.html`)
  - Table of Contents adjusted to include H1 titles (`config.toml`)
  - Removed permanent underline decoration from links (`_main.scss`)
  - Link color changed in post p tags to #0070F3 (`_single.scss`)
  - Cursor color changed to #0070F3 (`_logo.scss`)
  - blockquote color changed to #0070F3 (`_main.scss`)
  - Added avatar image to main page (avatar class added in `_main.scss`, logic to `index.html`)
  - Added last 5 posts to main page (`index.html`)
  - Pagination limit changed to 15 (was 10) (`config.toml`)
  - Footer layout changed slightly (`footer.html`)
  - Removed absolute path for cover images in posts (`single.html`)
