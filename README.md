# aaron notes

## 2023-12-20 deploy change log

changed deploy to Github Actions, as in [Host on GitHub Pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

## 2023-05-02 theme change log

- modified theme linked url: https://github.com/lijqhs/hugo-tranquilpeak-theme
- added Katex
  - [Math Typesetting in Hugo](https://mertbakir.gitlab.io/hugo/math-typesetting-in-hugo/)
- modified copyright footer (footer.html)
- favicon, ref: https://github.com/devcows/hugo-universal-theme/issues/69
- added **dark** mode, (add partial, merge css, merge js)
  - https://github.com/kakawait/hugo-tranquilpeak-theme/issues/384
  - https://github.com/AnceretMatthieu/hugo-tranquilpeak-theme
  - https://anceret-matthieu.fr/
- added `mermaid` support:
  - `mermaid: true`
  - see https://github.com/AnceretMatthieu/hugo-tranquilpeak-theme

add theme:
```shell
git submodule add https://github.com/lijqhs/hugo-tranquilpeak-theme.git themes/TranquilMod
```

update theme:
```shell
git submodule update --remote themes/TranquilMod
```

## resource reference
- [font awesome](https://fontawesome.com/v4/icons/)