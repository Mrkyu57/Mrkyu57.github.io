---
layout: default
title: "HOME"
---

<div class="container">

  <div class="main_text">   <h1>project:MK</h1>             
    <input class="search_bar" value="⌕" >    
  </div>
  
  <div class="menu_block">  
    <a href="/language_mainpost/" class="language-click">
      <h2>
        언어 language
      </h2> 
    </a>
    <a href="/math_mainpost/" class="math-click">
      <h2>
        수학 math
      </h2> 
    </a>
    <a href="/physics_mainpost/" class="physics-click">
      <h2>
        물리 physics
      </h2> 
    </a>
    <a href="/chemistry_mainpost/" class="chemistry-click">
      <h2>
        화학 chemistry
      </h2> 
    </a>
    <a href="/biology_mainpost/" class="biology-click">
      <h2>
        생물학 biology
      </h2> 
    </a>
    <a href="/economics_mainpost/" class="economics-click">
      <h2>
        경제학 economics
      </h2> 
    </a>
  
  </div>

</div>


<style>
  /* 전체 레이아웃을 좌/우로 나누는 설정 */
  .container {
    display: flex;
    align-items: center;       /* 좌측 요소와 우측 박스들의 세로 중앙 정렬 */
    justify-content: flex-start;
    gap: 100px;                /* 왼쪽 타이틀 세트와 오른쪽 박스 사이의 간격 */
    margin: 180px 120px;       /* 전체적인 화면 여백 */
  }
  
  .main_text {
    font-size: 2rem;
    margin: 0;                 /* container 여백을 사용하므로 초기화 */
  }      

  .search_bar {
    border: 1px solid #ccc;
    border-radius: 20px;
    padding: 8px;
    width: 200px;
    text-align: left;
    margin-top: 15px;          /* 타이틀 글씨와 검색바 사이 간격 */
  }
  
  /* 오른쪽 메뉴 블럭 (세로로 나란히 배열) */
  .menu_block {
    display: flex;
    flex-direction: column;   /* 수학, 물리를 위아래로 배치 */
    gap: 20px;                /* 수학 박스와 물리 박스 사이의 간격 */
  }

  /* 링크 기본 밑줄 제거 */
  .menu_block a {
    text-decoration: none;
  }

  .menu_block h2 {
    display: block;
    color: #ffffff;           
    padding: 25px 40px;       /* 가로로 긴 사각형 비율 조정 */
    border-radius: 10px;      
    width: 450px;             /* 사각형 박스 가로 길이 */
    box-sizing: border-box;   
    margin: 0;
    text-align: center;       
    font-size: 1.5rem;
    transition: transform 0.2s; /* 마우스 올렸을 때 부드러운 효과용 */
  }

  /* 마우스 올렸을 때 살짝 반응하는 효과 (선택 사항) */
  .menu_block h2:hover {
    transform: translateY(-3px);
  }

  .language-click h2 {
    background-color: #ce9154;
  }
  .math-click h2 {
    background-color: #5489ce;
  }

  .physics-click h2 {
    background-color: #8354ce;
  }
  .chemistry-click h2 {
    background-color: #cc342f;
  }
  .biology-click h2 {
    background-color: #2db662;
  }
  .economics-click h2 {
    background-color: #dda83f;
  }
</style>