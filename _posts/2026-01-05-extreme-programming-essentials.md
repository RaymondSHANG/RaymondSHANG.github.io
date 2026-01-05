---
layout: post
title: "Extreme Programming Essentials"
subtitle: "5 most important tips"
date: 2026-01-05 10:47:36
header-style: text
catalog: true
author: "Yuan"
tags: [XP, Agile, Tips, Extreme Programming, Test First, Pairing, Refactoring, Integration]
---
{% include linksref.html %}

Below I listed the 5 most important "XP" points for my reference during development:

- **Test First (TDD)** Write the test before the code. If it doesn't fail, don't write the logic.

- **Pair Up** Two eyes are better than one. Code with a partner for complex tasks to share knowledge and reduce bugs.

- **Keep It Simple (YAGNI)** You Aren't Gonna Need It. Do the simplest thing that could possibly work right now. Don't design for the future.

- **Refactor Mercilessly** Clean the code as you go. If it works but looks messy, fix the design immediately before moving on.

- **Integrate Often (CI)** Merge your code multiple times a day. Never let code sit on your machine for more than a few hours.

## A sticker friendly version:

<style>
  /* Use a specific container class instead of 'body' */
  .sticker-container {
    display: flex; 
    justify-content: center; 
    padding: 30px 0;
    width: 100%;
  }
  .sticker {
    background-color: #fff740; /* Post-it Yellow */
    width: 300px;
    padding: 20px;
    border: 1px solid #d3d3d3;
    box-shadow: 5px 5px 10px rgba(0,0,0,0.1);
    color: #333;
    font-family: "Courier New", Courier, monospace; 
    /* Rotate it slightly for a realistic look */
    transform: rotate(-1deg); 
  }
  .sticker h2 {
    margin-top: 0;
    text-align: center;
    border-bottom: 2px solid #333;
    padding-bottom: 10px;
    font-size: 22px;
    margin-bottom: 10px;
  }
  .sticker ul { padding-left: 20px; margin: 0; }
  .sticker li { margin-bottom: 8px; font-weight: bold; font-size: 16px; line-height: 1.2;}
  .sticker span { font-weight: normal; font-size: 14px; display: block; color: #555; margin-top: 2px;}
</style>

<div class="sticker-container">
  <div class="sticker">
    <h2>XP ESSENTIALS</h2>
    <ul>
      <li>1. Test First (TDD)
          <span>Write test. Fail. Code. Pass.</span></li>
      <li>2. Pair Up
          <span>Two eyes > One. Share the load.</span></li>
      <li>3. Keep It Simple (YAGNI)
          <span>Don't code for the future.</span></li>
      <li>4. Refactor Mercilessly
          <span>Clean code as you go.</span></li>
      <li>5. Integrate Often (CI)
          <span>Merge multiple times a day.</span></li>
    </ul>
  </div>
</div>

---
