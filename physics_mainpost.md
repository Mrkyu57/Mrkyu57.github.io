---
layout: post
title: "물리학"
koreantitle: 물리학
englishtitle: physics
info: 흠
color: "#8354ce"
permalink: /physics_mainpost/
---


<div class="physics_menu_block">  <!--선택메뉴-->

  <a href="/physics_mainpost_classicaldynamics/" class="classicaldynamics-click">
    <h2>
      고전역학 classical dynamics
    </h2> 
  </a>

  <a href="/physics_mainpost_fluidmechanics/" class="fluidmechanics-click">
    <h2>
      유체역학 fluid mechanics
    </h2> 
  </a>

  <a href="/physics_mainpost_thermodynamics/" class="thermodynamics-click">
    <h2>
      열역학 thermo dynamics
    </h2> 
  </a>

</div>


<style>



    .classicaldynamics-click h2{
        background-color: #8354ce;
        position: absolute;
        top: 400px;               
        left: 200px;
        margin: 0;
    }

    .fluidmechanics-click h2{
        background-color: #8354ce;
        position: absolute;
        top: 400px;               
        left: 400px;
        margin: 0;
    }
    .thermodynamics-click h2{
        background-color: #8354ce;
        position: absolute;
        top: 400px;               
        left: 600px;
        margin: 0;
    }
</style>

<div class="physics_menu_block">
  
  <div class="menu-row">
    <button class="main-btn" onclick="toggleSubMenu(this)">
      <h2>고전역학<br><span class="en-title">classical dynamics</span></h2>
    </button>
    <div class="sub-menu-bar hidden">
      <a href="/physics_logic_1/" class="sub-btn">
        <span class="ko-sub">운동학</span>
        <span class="en-sub">dot particle</span>
      </a>
      <a href="/physics_logic_2/" class="sub-btn">
        <span class="ko-sub">정역학</span>
        <span class="en-sub">statics</span>
      </a>
      <a href="/physics_logic_2/" class="sub-btn">
        <span class="ko-sub">동역학</span>
        <span class="en-sub">dynamics</span>
      </a>
    </div>
  </div>
  
  <div class="menu-row">
    <button class="main-btn" onclick="toggleSubMenu(this)">
      <h2>유체역학<br><span class="en-title">fluid mechanics</span></h2>
    </button>
    <div class="sub-menu-bar hidden">
      <a href="/physics_geometry_1/" class="sub-btn">
        <span class="ko-sub">유체정역학</span>
        <span class="en-sub">fluid statics</span>
      </a>
      <a href="/physics_geometry_2/" class="sub-btn">
        <span class="ko-sub">유체동역학</span>
        <span class="en-sub">fluid dynamics</span>
      </a>
    </div>
  </div>

  <div class="menu-row">
    <button class="main-btn" onclick="toggleSubMenu(this)">
      <h2>열역학<br><span class="en-title">thermo dynamics</span></h2>
    </button>
    <div class="sub-menu-bar hidden">
      <a href="/physics_algebra_1/" class="sub-btn">
        <span class="ko-sub">군론</span>
        <span class="en-sub">Group theory</span>
      </a>
      <a href="/physics_algebra_2/" class="sub-btn">
        <span class="ko-sub">선형대수학</span>
        <span class="en-sub">Linear algebra</span>
      </a>
    </div>
  </div>

  <div class="menu-row">
    <button class="main-btn" onclick="toggleSubMenu(this)">
      <h2>전자기학<br><span class="en-title">electromagnetism</span></h2>
    </button>
    <div class="sub-menu-bar hidden">
      <a href="/physics_algebra_1/" class="sub-btn">
        <span class="ko-sub">정전기학</span>
        <span class="en-sub">electrostatics</span>
      </a>
      <a href="/physics_algebra_2/" class="sub-btn">
        <span class="ko-sub">정자기학</span>
        <span class="en-sub">magnetostatics</span>
      </a>
    </div>
  </div>

</div>

<script>
  function toggleSubMenu(button) {
    const subMenu = button.nextElementSibling;
    
    // 현재 열려있는 다른 메뉴가 있다면 닫기 (아코디언 형태를 원하시면 이 부분 유지, 아니면 삭제)
    /*
    const allSubMenus = document.querySelectorAll('.sub-menu-bar');
    allSubMenus.forEach(menu => {
      if(menu !== subMenu) menu.classList.add('hidden');
    });
    */
    
    subMenu.classList.toggle('hidden');
  }
</script>

<style>
  /* 전체 컨테이너 여백 */
  .physics_menu_block {
    margin-top: 50px;
    padding: 20px;
  }

  /* 각 행(가로줄) 레이아웃 */
  .menu-row {
    display: flex;
    align-items: center;
    margin-bottom: 35px;
  }

  /* 메인 카테고리 버튼 */
  .main-btn {
    background-color: #8354ce; /* 스크린샷과 동일한 파란색 */
    color: #ffffff;
    border: none;
    cursor: pointer;
    width: 180px;
    height: 105px;
    border-radius: 12px;
    padding: 0 25px;
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
    justify-content: center;
    text-align: left;
    z-index: 2;
    box-shadow: 2px 4px 10px rgba(0, 0, 0, 0.15); /* 입체감을 위한 은은한 그림자 추가 */
    transition: transform 0.2s ease, box-shadow 0.2s ease;
    flex-shrink: 0;
  }

  .main-btn:hover {
    transform: translateY(-2px); /* 마우스 오버 시 살짝 위로 떠오름 */
    box-shadow: 2px 6px 14px rgba(0, 0, 0, 0.2);
  }

  .main-btn h2 {
    margin: 0;
    font-size: 1.4rem;
    font-weight: 700;
    line-height: 1.3;
    color: #ffffff;
    font-family: inherit;
  }

  .en-title {
    font-size: 1.1rem;
    font-weight: 400;
    opacity: 0.85; /* 영문명은 살짝 투명하게 처리하여 위계 분리 */
  }

  /* 서브 카테고리를 담는 회색 바 */
  .sub-menu-bar {
    background-color: #d4d4d4; /* 스크린샷 톤에 맞춘 회색 */
    height: 85px;
    display: flex;
    align-items: center;
    padding-left: 35px; 
    margin-left: -15px; /* 메인 버튼 뒤로 살짝 숨겨진 느낌 */
    border-radius: 0 8px 8px 0; /* 오른쪽 모서리만 둥글게 */
    z-index: 1;
    
    /* 애니메이션 설정 */
    max-width: 800px; /* 펼쳐졌을 때 충분한 길이 확보 */
    opacity: 1;
    transform: translateX(0);
    transition: all 0.4s cubic-bezier(0.25, 0.8, 0.25, 1); /* 부드럽고 텐션 있는 애니메이션 */
  }

  /* 숨겨진 상태일 때 */
  .sub-menu-bar.hidden {
    max-width: 0;
    opacity: 0;
    padding-left: 0;
    transform: translateX(-30px);
    pointer-events: none;
    overflow: hidden; /* 영역 밖 글자 숨김 */
  }

  /* 서브 카테고리 버튼 */
  .sub-btn {
    background-color: #8354ce;
    color: #ffffff;
    text-decoration: none;
    height: 100%;
    min-width: 150px; /* 글자가 길어도 깨지지 않게 최소 너비 설정 */
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: flex-start; /* 텍스트 왼쪽 정렬 */
    padding: 0 20px;
    margin-right: 25px;
    box-sizing: border-box;
    transition: background-color 0.2s ease;
  }
  
  .sub-btn:hover {
    background-color: #4a7ab8; /* 마우스 오버 시 살짝 어두워지는 피드백 */
  }

  /* 세부 과목 한글 텍스트 */
  .ko-sub {
    font-size: 1.1rem;
    font-weight: 700;
    white-space: nowrap;
    margin-bottom: 3px;
  }

  /* 세부 과목 영문 텍스트 */
  .en-sub {
    font-size: 0.85rem;
    font-weight: 400;
    white-space: nowrap;
    opacity: 0.8;
  }
</style>