---
layout: single
title: "RPS Demo"
permalink: /rps/
author_profile: false
# put your current tunnel URL here so it's easy to update
api_base: "https://slides-recovery-samba-financing.trycloudflare.com"
---

<div>
  <h1>Rock–Paper–Scissors</h1>
  <button onclick="play('rock')">Rock</button>
  <button onclick="play('paper')">Paper</button>
  <button onclick="play('scissors')">Scissors</button>
  <pre id="out" style="margin-top:1rem; white-space:pre-wrap;"></pre>
</div>

<script>
  // Jekyll injects the URL from front matter:
  const API_BASE = "{{ page.api_base }}";

  async function play(move){
    try {
      const res = await fetch(`${API_BASE}/play`, {
        method: 'POST',
        headers: {'Content-Type':'application/json'},
        body: JSON.stringify({ user_move: move })
      });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      document.getElementById('out').textContent =
        JSON.stringify(data, null, 2);
    } catch (e) {
      document.getElementById('out').textContent =
        `Request failed. Is the tunnel URL current?\n\n${e}`;
    }
  }
</script>
