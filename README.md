# 21st Century Weather Software Wiki

https://21centuryweather.github.io/21st-Century-Weather-Software-Wiki/intro.html

This wiki is built with [Quarto](https://quarto.org/). Install Quarto, then run:

```bash
quarto render --no-execute
quarto preview --no-execute
```

The rendered site is written to `_site/`. Notebook pages use their saved outputs, so
notebooks that require Gadi data can be updated on Gadi and committed without
running them during the site build. The GitHub Actions workflow checks the site
on `quarto_test` and publishes it from `main` when changes are pushed there.
