---
title: A post with a table
description: Ein Post wo ich mit Tables rumspiele
date: '2020-02-20'
tags:
  - demo-content
  - code
  - blog
---

In diesem Post probiere ich zuerst einmal, eine Tabelle so zu gestalten, dass die nicht über den Content hinausragt (und stattdessen gescrollt werden kann).

---

<script>
  function fullWidthTable() {
  let table = document.querySelector(".reel");
  let classes = table.classList;
  if (classes.contains("full-width")) {
  classes.remove("full-width");
  } else {
    classes.add("full-width");
  }
}
</script>

<button onClick="fullWidthTable()">Toggle full width</button>

<div class="reel">
<table>
  <tr>
    <th>Header 1</th>
    <th>Header 2</th>
    <th>Header 3</th>
    <th>Header 4</th>
    <th>Noch ein Header</th>
    <th>Header 6</th>
    <th>Header 7</th>
  </tr>
  <tr>
    <td style="width: 8rem;">1</td>
    <td>Hier steht jetzt ein etwas längerer Text drin. Das kann z.B. ein Prozess schritt sein oder eine Beschreibung.</td>
    <td>Hier ist nur ein Verantwortlicher</td>
    <td>Und ein Anhang</td>
    <td>Noch etwas mehr</td>
    <td>Even moar content!!!</td>
    <td>Und zum Schluss noch das hier.</td>
  </tr>
  <tr>
    <td>2<br></td>
    <td>Und noch ein Text. Dieses mal ein wenig kürzer als der andere.</td>
    <td>Person 3</td>
    <td>Anhang 1</td>
    <td></td>
    <td>Inhalt</td>
  </tr>
  <tr>
    <td>3</td>
    <td>Ein ganz kurzer Text</td>
    <td>Person 1</td>
    <td></td>
  </tr>
</table>
</div>

This is a table. We'll see how this gets rendered.
