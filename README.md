# xcomadvancedwidth
# this does not work. I am learning!

// ==UserScript==
// @name         X.com Clean & Wide Layout (Debug)
// @namespace    https://github.com/yourname/x-clean-wide
// @version      2.2
// @description  Debug version to resize center tweet column with gutters for X.com
// @match        https://x.com/*
// @match        https://www.x.com/*
// @match        https://mobile.x.com/*
// @run-at       document-idle
// ==/UserScript==

(function () {
  'use strict';

  // === [0] Exclude Specific Pages ===
  const excludedPaths = ['/messages', '/settings'];
  if (excludedPaths.some(path => location.pathname.startsWith(path))) {
    console.log('[X.com Script] Excluded path, script not running');
    return;
  }

  // === [1] Default Settings ===
  const targetWidth = 1000; // Fixed width for testing (px)
  const gutterWidth = 20; // Gutter size (px)

  // === [2] Inject CSS Styles ===
  function injectStyles(width) {
    const styleId = 'x-clean-wide-styles';
    if (document.getElementById(styleId)) return;

    const style = document.createElement('style');
    style.id = styleId;
    style.textContent = `
      main[role="main"] {
        width: 100% !important;
        max-width: none !important;
        margin: 0 auto !important;
        display: flex !important;
        justify-content: center !important;
        background: rgba(0, 0, 255, 0.1) !important; /* Debug: Blue tint for main */
      }
      .x-left-column {
        background: #e6f3ff !important; /* Debug: Light blue for left nav */
      }
      .x-center-column {
        width: ${width}px !important;
        max-width: ${width}px !important;
        min-width: ${width}px !important;
        margin: 0 ${gutterWidth}px !important;
        flex: 0 0 auto !important;
        background: #ffff99 !important; /* Debug: Yellow for center tweets */
      }
      .x-right-column {
        background: #ff9999 !important; /* Debug: Light red for right sidebar */
      }
      article[role="article"] > div {
        width: ${width}px !important;
        max-width: ${width}px !important;
        margin: 0 auto !important;
      }
      nav[role="navigation"] {
        position: fixed !important;
        top: 0 !important;
        left: ${gutterWidth}px !important;
        z-index: 999 !important;
        background: #e6f3ff !important; /* Debug: Light blue for nav */
      }
    `;
    document.head.appendChild(style);
    console.log('[X.com Script] Styles injected with width:', width);
  }

  // === [3] Log DOM Hierarchy ===
  function logHierarchy(element, label, depth = 3) {
    const hierarchy = [];
    let current = element;
    for (let i = 0; i < depth && current; i++) {
      hierarchy.push({
        tag: current.tagName,
        classes: current.className,
        style: getComputedStyle(current).display,
        childrenCount: current.children.length,
        id: current.id || 'none',
        dataTestid: current.dataset.testid || 'none'
      });
      current = current.parentElement;
    }
    console.log(`[X.com Script] ${label} hierarchy:`, hierarchy);
  }

  // === [4] Dynamic Resize Function ===
  function resizeLayout() {
    console.log('[X.com Script] Running resizeLayout...');

    try {
      // (a) Main wrapper
      const mainWrapper = document.querySelector('main[role="main"]');
      if (!mainWrapper) {
        console.warn('[X.com Script] Main wrapper not found');
        return;
      }
      logHierarchy(mainWrapper, 'Main wrapper');

      // (b) Find layout wrapper
      let layoutWrapper = null;
      const article = document.querySelector('article[role="article"]');
      if (article) {
        let parent = article;
        for (let i = 0; i < 5; i++) {
          parent = parent.parentElement;
          if (parent && getComputedStyle(parent).display === 'flex' && parent.children.length >= 2) {
            layoutWrapper = parent;
            break;
          }
        }
        logHierarchy(article, 'Article');
      }

      if (!layoutWrapper) {
        const flexContainers = Array.from(document.querySelectorAll('div')).filter(div =>
          getComputedStyle(div).display === 'flex' && div.children.length >= 2 && div.closest('main[role="main"]')
        );
        layoutWrapper = flexContainers[0];
      }

      if (!layoutWrapper) {
        const nav = document.querySelector('nav[role="navigation"]');
        if (nav) {
          layoutWrapper = nav.parentElement?.parentElement;
          logHierarchy(nav, 'Navigation');
        }
      }

      if (!layoutWrapper) {
        console.warn('[X.com Script] Layout wrapper not found. Main wrapper children:', 
          Array.from(mainWrapper.children).map((el, i) => ({
            index: i,
            tag: el.tagName,
            classes: el.className,
            style: getComputedStyle(el).display,
            childrenCount: el.children.length,
            dataTestid: el.dataset.testid || 'none'
          }))
        );
        return;
      }

      // (c) Identify columns
      const columns = Array.from(layoutWrapper.children);
      let leftCol = null, centerCol = null, rightCol = null;

      // Find center column by checking for article[role="article"]
      columns.forEach((col, i) => {
        if (col.querySelector('article[role="article"]')) {
          centerCol = col;
        } else if (col.querySelector('nav[role="navigation"]')) {
          leftCol = col;
        } else if (col.dataset.testid === 'sidebarColumn') {
          rightCol = col;
        }
      });

      // Fallback: Assume second column is center if no articles found
      if (!centerCol && columns.length >= 2) {
        centerCol = columns[1];
      }

      if (!centerCol) {
        console.warn('[X.com Script] Center column not found. Layout wrapper children:', 
          columns.map((el, i) => ({
            index: i,
            tag: el.tagName,
            classes: el.className,
            style: getComputedStyle(el).display,
            childrenCount: el.children.length,
            dataTestid: el.dataset.testid || 'none',
            hasArticle: !!el.querySelector('article[role="article"]')
          }))
        );
        return;
      }

      // (d) Add classes to columns
      if (leftCol) leftCol.classList.add('x-left-column');
      centerCol.classList.add('x-center-column');
      if (rightCol) rightCol.classList.add('x-right-column');

      console.log('[X.com Script] Columns identified:', {
        leftCol: leftCol ? leftCol.className : 'none',
        centerCol: centerCol.className,
        rightCol: rightCol ? rightCol.className : 'none'
      });

      // (e) Apply styles to layout wrapper
      Object.assign(layoutWrapper.style, {
        display: 'flex !important',
        gap: `${gutterWidth}px !important`,
        justifyContent: 'center !important',
        padding: `0 ${gutterWidth}px !important`,
        background: 'rgba(0, 255, 0, 0.1) !important' // Debug: Light green
      });

      console.log('[X.com Script] Layout updated successfully');
    } catch (error) {
      console.error('[X.com Script] Resize error:', error.message);
    }
  }

  // === [5] Initialize with Retry ===
  function initialize(attempt = 1, maxAttempts = 30) {
    console.log(`[X.com Script] Initialize attempt ${attempt}/${maxAttempts}`);
    if (document.readyState === 'complete' || document.readyState === 'interactive') {
      injectStyles(targetWidth);
      resizeLayout();
      const observer = new MutationObserver(resizeLayout);
      observer.observe(document.body, { childList: true, subtree: true });
      window.addEventListener('resize', resizeLayout);
      window.addEventListener('beforeunload', () => observer.disconnect());
    } else if (attempt < maxAttempts) {
      setTimeout(() => initialize(attempt + 1, maxAttempts), 300);
    } else {
      console.error('[X.com Script] Failed to initialize: DOM not ready after max attempts');
    }
  }

  // Start initialization
  initialize();
})();
