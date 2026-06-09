// ==UserScript==
// @name         Instagram Auto Unlike Bot (5.0)
// @namespace    http://tampermonkey.net/
// @version      5.0
// @description  Auto-unlike Instagram posts/reels — resumes after reload
// @author       Marco / fixed
// @match        https://www.instagram.com/your_activity/interactions/likes/
// @grant        none
// @run-at       document-idle
// ==/UserScript==

(async function () {
  'use strict';

  // ── CONFIG ──────────────────────────────────────────────
  const BATCH_SIZE      = 80;
  const SCROLL_ATTEMPTS = 6;
  const DELAY_SCROLL    = 1200;
  const DELAY_CHECKBOX  = 60;
  const MAX_SELECT_TRIES = 5;
  // ────────────────────────────────────────────────────────

  const delay = ms => new Promise(r => setTimeout(r, ms));

  // Status overlay so you can see progress without opening DevTools
  const statusEl = document.createElement('div');
  Object.assign(statusEl.style, {
    position: 'fixed', bottom: '20px', right: '20px', zIndex: '99999',
    background: '#111', color: '#0f0', fontFamily: 'monospace',
    fontSize: '13px', padding: '10px 16px', borderRadius: '8px',
    maxWidth: '320px', lineHeight: '1.5', pointerEvents: 'none',
    boxShadow: '0 2px 12px rgba(0,0,0,0.5)'
  });
  document.body.appendChild(statusEl);
  const log = msg => {
    console.log(msg);
    statusEl.textContent = msg;
  };

  const realClick = async el => {
    if (!el) return;
    el.scrollIntoView({ block: 'center' });
    await delay(300);
    const r = el.getBoundingClientRect();
    const x = r.left + r.width / 2, y = r.top + r.height / 2;
    ['pointerdown','mousedown','pointerup','mouseup','click'].forEach(t =>
      el.dispatchEvent(new MouseEvent(t, { bubbles: true, cancelable: true, clientX: x, clientY: y }))
    );
    await delay(500);
  };

  const scrollCollection = async () => {
    const c = document.querySelector('[data-bloks-name="bk.components.Collection"]');
    if (!c) return;
    let prev = 0;
    for (let i = 0; i < SCROLL_ATTEMPTS; i++) {
      c.scrollBy(0, c.scrollHeight);
      await delay(DELAY_SCROLL);
      if (c.scrollHeight === prev) break;
      prev = c.scrollHeight;
    }
  };

  const findUnlikeButton = () => {
    const span = Array.from(document.querySelectorAll('span')).find(s => {
      const t = s.textContent.trim();
      if (t !== 'Unlike' && t !== 'Non mi piace più') return false;
      const r = s.getBoundingClientRect();
      return r.width > 0 && r.height > 0;
    });
    if (!span) return null;
    let el = span;
    while (el) {
      if (el.getAttribute?.('role') === 'button') {
        const r = el.getBoundingClientRect();
        if (r.width > 0 && r.height > 0) return el;
      }
      el = el.parentElement;
    }
    return null;
  };

  const clickUnlike = async () => {
    for (let i = 0; i < 6; i++) {
      const btn = findUnlikeButton();
      if (btn) { log('Unlike button found'); await realClick(btn); return true; }
      log(`Waiting for Unlike button… (${i + 1}/6)`);
      await delay(700);
    }
    log('Unlike button not found');
    return false;
  };

  const findConfirmButton = () =>
    Array.from(document.querySelectorAll('button')).find(b => {
      if (b.innerText.trim() !== 'Unlike') return false;
      const r = b.getBoundingClientRect();
      return r.width > 0 && r.height > 0;
    });

  const clickConfirm = async () => {
    for (let i = 0; i < 6; i++) {
      const btn = findConfirmButton();
      if (btn) {
        log('Confirming unlike…');
        btn.scrollIntoView({ block: 'center' });
        await delay(200);
        btn.click();
        await delay(800);
        return true;
      }
      log(`Waiting for confirm button… (${i + 1}/6)`);
      await delay(500);
    }
    log('Confirm button not found');
    return false;
  };

  const waitForText = async (texts, timeout = 8000) => {
    const end = Date.now() + timeout;
    while (Date.now() < end) {
      const el = Array.from(document.querySelectorAll('[role="button"], button, span'))
        .find(e => texts.some(t => (e.innerText || '').trim() === t));
      if (el) return el;
      await delay(200);
    }
    return null;
  };

  // ── MAIN ────────────────────────────────────────────────
  const run = async () => {
    try {
      const isRestart = sessionStorage.getItem('unlikeBot') === 'running';

      if (isRestart) {
        log('Resuming after reload…');
        await delay(2000); // let page settle
      } else {
        log('Unlike bot starting…');
        sessionStorage.setItem('unlikeBot', 'running');
      }

      // Find "Select" button
      let select = null;
      for (let i = 0; i < MAX_SELECT_TRIES; i++) {
        select = await waitForText(['Select', 'Seleziona'], 3000);
        if (select) break;
        log(`Select not found, retrying… (${i + 1}/${MAX_SELECT_TRIES})`);
      }

      if (!select) {
        log('Select not found — reloading…');
        location.reload();
        return;
      }

      log('Clicking Select…');
      await realClick(select);
      await delay(600);

      log('Scrolling to load items…');
      await scrollCollection();

      // Tick checkboxes
      const boxes = document.querySelectorAll('[aria-label="Toggle checkbox"]');
      const count = Math.min(BATCH_SIZE, boxes.length);
      log(`Selecting ${count} items…`);

      for (let i = 0; i < count; i++) {
        boxes[i].click();
        await delay(DELAY_CHECKBOX);
      }

      if (count === 0) {
        log('No items left — done!');
        sessionStorage.removeItem('unlikeBot');
        return;
      }

      await delay(1200);
      await clickUnlike();
      await delay(800);
      await clickConfirm();

      log('Batch done — reloading for next batch…');
      await delay(3000);
      // sessionStorage key stays 'running' → auto-resume on reload
      location.reload();

    } catch (e) {
      console.error(e);
      log('Error — retrying in 3s…');
      await delay(3000);
      run();
    }
  };

  run();
})();



