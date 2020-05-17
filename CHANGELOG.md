# Changelog

All notable changes to this project will be documented in this file.

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