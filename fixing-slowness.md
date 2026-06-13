### Why it felt slow

The preloader was tied to **`window`’s `load` event**. That event runs only after **everything** the browser treats as part of the page has finished, including:

- All images (you have a large portfolio section)
- Google Fonts CSS and font files
- Other stylesheets and scripts

So the spinner was not “your JS being slow”; it was **waiting on the slowest asset**.

Your `index.html` also loads `main.js` with **`async`**, so the script can run **after** `load` has already fired. In that case a `load` listener **never runs**, which can leave the preloader up indefinitely (another failure mode besides “slow”).


### What we changed

In `main.js` the preloader logic now:

The smallest meaningful change is **stop using `window` `load`** and hide the preloader when the **DOM is ready** (`DOMContentLoaded`), plus **`readyState !== 'loading'`** so it still works when `main.js` runs late with `async`.

That’s what’s in the file now:

1. **stops using `window` `load`** and hide the preloader when the **DOM is ready** (`DOMContentLoaded`), plus **`readyState !== 'loading'`** so it still works when `main.js` runs late with `async`.


```48:61:/Users/dave/king/My-Graphic-Design-Portfolio/assets/js/main.js
  /**
   * Preloader — use DOM ready, not window "load" (load waits for every image/font).
   */
  const preloader = document.querySelector('#preloader');
  if (preloader) {
    const hide = () => preloader.remove();
    if (document.readyState === 'loading') {
      document.addEventListener('DOMContentLoaded', hide, { once: true });
    } else {
      hide();
    }
  }
```


**Tradeoff:** Images/fonts may still be loading when the overlay goes away, so you might briefly see layout or images filling in. If you’d rather cap wait time instead, we can put back a short `setTimeout` race with `load`—slightly more code, different tradeoff.

**Optional follow-ups** (not applied): host critical fonts locally or subset them, compress/lazy-load portfolio images.