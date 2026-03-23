---
layout: default
title: "수학"
permalink: /mathmainpost/
---

<div class="math_background">
    <h1 class="math_title">
        <span class="math_korean_name">수학</span>
        <span class="math_english_name">math</span>
    </h1>
    <p class="math_info">공식과 논리로 세상을 해석하는 기록들</p>
</div>

<div class="math_menu_block">  <!--선택메뉴-->

  <a href="/logicmainpost/" class="logic-click">
    <h2>
      논리학 logic
    </h2> 
  </a>

  <a href="/algebramainpost/" class="algebra-click">
    <h2>
      대수학 algebra
    </h2> 
  </a>

  <a href="/analysismainpost/" class="analysis-click">
    <h2>
      해석학 analysis
    </h2> 
  </a>

</div>


<style>
    /* 전체 배경 박스 */
    .math_background {
        background-color: #5489ce; 
        margin: -30px -20px 50px -20px; 
        padding: 80px 40px; 
        color: white;
        text-align: left;
    }

    .math_title {
        margin: 0;
        line-height: 1.2;
    }

    .math_korean_name {
        font-size: 3.5rem; 
        font-weight: 800; 
        letter-spacing: -2px;
        margin-right: 10px;
    }

    .math_english_name {
        font-size: 1.5rem; 
        font-weight: 300;
        opacity: 0.8;
    }

    .math_info {
        margin-top: 15px;
        font-size: 1.1rem;
        opacity: 0.9;
    }

    .math_menu_block h2 {
        display: inline-block;
        color: #ffffff;           
        padding: 40px;
        border-radius: 10px;      
        text-decoration: none;
        width: 80px;
    }
    .logic-click h2{
        background-color: #5489ce;
        position: absolute;
        top: 400px;               
        left: 200px;
        margin: 0;
    }

    .algebra-click h2{
        background-color: #5489ce;
        position: absolute;
        top: 400px;               
        left: 400px;
        margin: 0;
    }
    .analysis-click h2{
        background-color: #5489ce;
        position: absolute;
        top: 400px;               
        left: 600px;
        margin: 0;
    }
</style>