---
title: A post with a table
date: '2020-02-20'
tags:
  - demo-content
  - code
  - blog
---

The best way to demo a code post is to display a real life post, so check out this one from [andy-bell.design](https://andy-bell.design/wrote/creating-a-full-bleed-css-utility/) about a full bleed CSS utility.

---

<script>
  function fullWidthTable() {
  let table = document.querySelector("table");
  let classes = table.classList;
  if (classes.contains("full-width")) {
  classes.remove("full-width");
  } else {
    classes.add("full-width");
  }
}
</script>

<button onClick="fullWidthTable()">Toggle full width</button>

<div class="oveflow-wrapper">
<table class="data-table">
  <tr>
    <th>Header 1</th>
    <th>Header 2</th>
    <th>Header 3</th>
    <th>Header 4</th>
    <th>Noch ein Header</th>
  </tr>
  <tr>
    <td>1</td>
    <td>Hier steht jetzt ein etwas längerer Text drin. Das kann z.B. ein Prozess schritt sein oder eine Beschreibung.</td>
    <td>Hier ist nur ein Verantwortlicher</td>
    <td>Und ein Anhang</td>
    <td>Noch etwas mehr</td>
  </tr>
  <tr>
    <td>2<br></td>
    <td>Und noch ein Text. Dieses mal ein wenig kürzer als der andere.</td>
    <td>Person 3</td>
    <td>Anhang 1</td>
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
