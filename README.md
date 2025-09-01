<!doctype html>
<html lang="ko" class="scroll-smooth">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>대한민국 상급종합병원 디렉터리</title>
  <meta name="description" content="상급종합병원 47개 병원명·주소·대표번호 한 페이지 정리. 클릭 한 번으로 전화·지도·공유 가능." />
  <!-- TailwindCSS (Play CDN) -->
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = { darkMode: 'class' };
  </script>
  <!-- Lucide 아이콘 -->
  <script src="https://unpkg.com/lucide@latest/dist/umd/lucide.min.js"></script>
  <style>
    .card{box-shadow:0 1px 2px rgba(16,24,40,.04),0 1px 3px rgba(16,24,40,.1)}
    .btn{transition:all .15s}
  </style>
</head>
<body class="min-h-screen bg-gradient-to-b from-sky-50 via-white to-white text-slate-800 dark:bg-slate-900 dark:text-slate-100">
  <!-- 헤더 -->
  <header class="sticky top-0 z-40 backdrop-blur bg-white/70 dark:bg-slate-900/70 border-b border-slate-200/60 dark:border-slate-800">
    <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8 py-3 flex items-center gap-3">
      <div class="flex-1">
        <h1 class="text-xl sm:text-2xl font-extrabold tracking-tight">대한민국 상급종합병원 디렉터리</h1>
        <p class="text-xs sm:text-sm text-slate-500 dark:text-slate-400">병원명·주소·대표번호를 한눈에, 클릭 한 번으로 전화·지도·공유</p>
      </div>
      <div class="flex items-center gap-2">
        <button id="modeBtn" class="btn inline-flex items-center gap-2 rounded-full border border-slate-200 dark:border-slate-700 px-3 py-2 text-sm hover:shadow-sm">
          <i data-lucide="moon" class="w-4 h-4"></i>
          <span class="hidden sm:inline">다크 모드</span>
        </button>
        <button id="csvDownBtn" class="btn inline-flex items-center gap-2 rounded-full border border-slate-200 dark:border-slate-700 px-3 py-2 text-sm hover:shadow-sm">
          <i data-lucide="download" class="w-4 h-4"></i>
          <span class="hidden sm:inline">CSV 다운로드</span>
        </button>
        <button id="csvUpBtn" class="btn inline-flex items-center gap-2 rounded-full bg-sky-600 text-white px-3 py-2 text-sm hover:bg-sky-700">
          <i data-lucide="upload" class="w-4 h-4"></i>
          <span class="hidden sm:inline">CSV 불러오기</span>
        </button>
        <button id="shareBtn" class="btn inline-flex items-center gap-2 rounded-full border border-slate-200 dark:border-slate-700 px-3 py-2 text-sm hover:shadow-sm">
          <i data-lucide="share-2" class="w-4 h-4"></i>
          <span class="hidden sm:inline">공유</span>
        </button>
        <button id="resetBtn" title="초기 데이터로 복원" class="btn hidden sm:inline-flex items-center gap-2 rounded-full border border-slate-200 dark:border-slate-700 px-3 py-2 text-sm hover:shadow-sm">
          <i data-lucide="rotate-ccw" class="w-4 h-4"></i>
        </button>
      </div>
    </div>
  </header>

  <!-- 검색/필터 -->
  <section class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8 py-6">
    <div class="flex flex-col gap-3 sm:flex-row sm:items-center sm:justify-between">
      <div class="relative flex-1">
        <i data-lucide="search" class="w-5 h-5 absolute left-3 top-1/2 -translate-y-1/2 text-slate-400"></i>
        <input id="searchInput" type="text" placeholder="병원명, 주소, 지역, 전화번호로 검색" class="w-full rounded-xl border border-slate-200 dark:border-slate-700 bg-white/70 dark:bg-slate-900/70 py-3 pl-11 pr-4 outline-none focus:ring-2 focus:ring-sky-500">
      </div>
      <div id="regionWrap" class="flex flex-wrap gap-2 mt-1"></div>
    </div>
    <div class="mt-3 text-xs sm:text-sm text-slate-500 dark:text-slate-400">
      <p>※ 본 목록은 보건복지부 제5기(2024~2026) 상급종합병원 <span class="font-semibold">총 47개</span>를 기준으로 구성되었습니다. 병원별 대표번호는 콜센터/대표전화 유형에 따라 다를 수 있습니다.</p>
    </div>
  </section>

  <!-- 리스트 -->
  <main class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8 pb-24">
    <div id="list" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4"></div>
    <div id="empty" class="hidden text-center text-slate-500 dark:text-slate-400 py-20">조건에 맞는 병원이 없습니다.</div>
  </main>

  <!-- CSV 임포터 모달 -->
  <div id="importModal" class="hidden fixed inset-0 z-50 bg-black/50 items-center justify-center p-4">
    <div class="w-full max-w-3xl rounded-2xl bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800">
      <div class="p-5 border-b border-slate-200 dark:border-slate-800">
        <h2 class="text-lg font-bold">CSV로 병원 목록 불러오기</h2>
        <p class="text-sm text-slate-500 dark:text-slate-400 mt-1">열 순서: <code>name, region, address, phone, website</code>. 첫 줄은 반드시 헤더여야 합니다.</p>
        <p class="text-xs text-slate-400 mt-1">예: <code>서울아산병원,서울,서울특별시 송파구 올림픽로43길 88,1688-7575,https://www.amc.seoul.kr</code></p>
      </div>
      <div class="p-5">
        <textarea id="csvTextarea" rows="10" placeholder="name,region,address,phone,website&#10;서울아산병원,서울,서울특별시 송파구 올림픽로43길 88,1688-7575,https://www.amc.seoul.kr&#10;삼성서울병원,서울,서울특별시 강남구 일원로 81,1599-3114,https://www.samsunghospital.com" class="w-full rounded-xl border border-slate-200 dark:border-slate-700 bg-white/80 dark:bg-slate-900/70 p-3 font-mono text-xs"></textarea>
        <div class="mt-4 flex items-center justify-end gap-2">
          <button id="modalCloseBtn" class="rounded-full px-4 py-2 text-sm border border-slate-200 dark:border-slate-700">닫기</button>
          <button id="modalImportBtn" class="rounded-full px-4 py-2 text-sm bg-sky-600 text-white hover:bg-sky-700">불러오기</button>
        </div>
      </div>
    </div>
  </div>

  <!-- 푸터 -->
  <footer class="border-t border-slate-200 dark:border-slate-800 py-8 mt-10">
    <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8 text-xs sm:text-sm text-slate-500 dark:text-slate-400 leading-relaxed">
      <p class="mb-1">※ 본 페이지는 상급종합병원 연락처/주소를 한눈에 보고 전화·지도로 바로 연결하도록 설계되었습니다. (공유 버튼으로 링크 전송 가능)</p>
      <p>※ 데이터 최신성 확보를 위해 정기적으로 CSV 업데이트를 권장합니다. (보건복지부 고시·각 병원 홈페이지 기준)</p>
    </div>
  </footer>

  <script>
  // ===== 데이터 (47개) =====
  const initialHospitals = [
    // ——— 서울권 (14) ———
    { name: "강북삼성병원", region: "서울", address: "서울특별시 종로구 새문안로 29", phone: "1599-8114", website: "https://www.kbsmc.co.kr" },
    { name: "건국대학교병원", region: "서울", address: "서울특별시 광진구 능동로 120-1", phone: "1588-1533", website: "https://www.kuh.ac.kr" },
    { name: "경희대학교병원", region: "서울", address: "서울특별시 동대문구 경희대로 23", phone: "02-958-8114", website: "https://www.khmc.or.kr" },
    { name: "고려대학교구로병원", region: "서울", address: "서울특별시 구로구 구로동로 148", phone: "1577-9966", website: "https://guro.kumc.or.kr" },
    { name: "고려대학교안암병원", region: "서울", address: "서울특별시 성북구 고려대로 73", phone: "1577-0083", website: "https://anam.kumc.or.kr" },
    { name: "삼성서울병원", region: "서울", address: "서울특별시 강남구 일원로 81", phone: "1599-3114", website: "https://www.samsunghospital.com" },
    { name: "서울대학교병원", region: "서울", address: "서울특별시 종로구 대학로 101", phone: "1588-5700", website: "https://www.snuh.org" },
    { name: "연세대학교 강남세브란스병원", region: "서울", address: "서울특별시 강남구 언주로 211", phone: "1599-6114", website: "https://gs.severance.healthcare" },
    { name: "연세대학교 세브란스병원(신촌)", region: "서울", address: "서울특별시 서대문구 연세로 50-1", phone: "1599-1004", website: "https://sev.severance.healthcare" },
    { name: "이화여자대학교 서울병원", region: "서울", address: "서울특별시 강서구 공항대로 260", phone: "1522-7000", website: "https://seoul.eumc.ac.kr" },
    { name: "재단법인 아산사회복지재단 서울아산병원", region: "서울", address: "서울특별시 송파구 올림픽로43길 88", phone: "1688-7575", website: "https://www.amc.seoul.kr" },
    { name: "중앙대학교병원", region: "서울", address: "서울특별시 동작구 흑석로 102", phone: "1800-1114", website: "https://www.cauhs.or.kr" },
    { name: "학교법인 가톨릭학 가톨릭대학교 서울성모병원", region: "서울", address: "서울특별시 서초구 반포대로 222", phone: "1588-1511", website: "https://www.cmcseoul.or.kr" },
    { name: "한양대학교병원", region: "서울", address: "서울특별시 성동구 왕십리로 222-1", phone: "02-2290-8114", website: "https://www.hyumc.com" },

    // ——— 경기 서북부권 (4) ———
    { name: "가톨릭대학교 인천성모병원", region: "경기·인천 서북부", address: "인천광역시 부평구 동수로 56", phone: "1544-9004", website: "https://www.cmcism.or.kr" },
    { name: "순천향대학교 부천병원", region: "경기·인천 서북부", address: "경기도 부천시 조마루로 170", phone: "032-621-5114", website: "https://www.schmc.ac.kr/bucheon" },
    { name: "의료법인 길의료재단 길병원(가천대 길병원)", region: "경기·인천 서북부", address: "인천광역시 미추홀구 길병원로 21", phone: "1577-2299", website: "https://gh.gachon.ac.kr" },
    { name: "인하대학교병원", region: "경기·인천 서북부", address: "인천광역시 중구 인항로 27", phone: "032-890-2114", website: "https://www.inha.com" },

    // ——— 경기 남부권 (5) ———
    { name: "가톨릭대학교 성빈센트병원", region: "경기 남부", address: "경기도 수원시 팔달구 중부대로 93", phone: "1577-8588", website: "https://www.stvincent.or.kr" },
    { name: "고려대학교 안산병원", region: "경기 남부", address: "경기도 안산시 단원구 적금로 123", phone: "1577-0032", website: "https://ansan.kumc.or.kr" },
    { name: "분당서울대학교병원", region: "경기 남부", address: "경기도 성남시 분당구 구미로173번길 82", phone: "1588-3369", website: "https://www.snubh.org" },
    { name: "아주대학교병원", region: "경기 남부", address: "경기도 수원시 영통구 월드컵로 164", phone: "1688-6114", website: "https://hosp.ajoumc.or.kr" },
    { name: "한림대학교 성심병원(평촌)", region: "경기 남부", address: "경기도 안양시 동안구 평촌로 170", phone: "1577-1801", website: "https://www.hallym.or.kr/hallymuniv_sub" },

    // ——— 강원권 (2) ———
    { name: "강릉아산병원", region: "강원", address: "강원특별자치도 강릉시 사천면 방동길 38", phone: "033-610-3114", website: "https://gangneungamc.or.kr" },
    { name: "연세대학교 원주세브란스기독병원", region: "강원", address: "강원특별자치도 원주시 일산로 20", phone: "033-741-0114", website: "https://www.ywmc.or.kr" },

    // ——— 충북권 (1) ———
    { name: "충북대학교병원", region: "충북", address: "충청북도 청주시 서원구 1순환로 776", phone: "043-269-6114", website: "https://www.cbnuh.or.kr" },

    // ——— 충남권 (3) ———
    { name: "단국대학교병원(천안)", region: "충남·대전", address: "충청남도 천안시 동남구 망향로 201", phone: "1588-0063", website: "https://www.dkuh.co.kr" },
    { name: "충남대학교병원", region: "충남·대전", address: "대전광역시 중구 문화로 282", phone: "042-280-7114", website: "https://www.cnuh.co.kr" },
    { name: "건양대학교병원", region: "충남·대전", address: "대전광역시 서구 관저동로 158", phone: "1577-3330", website: "https://www.kyuh.ac.kr" },

    // ——— 전북권 (2) ———
    { name: "원광대학교병원", region: "전북", address: "전라북도 익산시 무왕로 895", phone: "063-859-2114", website: "https://www.wkuh.org" },
    { name: "전북대학교병원", region: "전북", address: "전라북도 전주시 덕진구 건지로 20", phone: "1577-7877", website: "https://www.jbuh.co.kr" },

    // ——— 전남권 (3) ———
    { name: "전남대학교병원", region: "광주·전남", address: "광주광역시 동구 제봉로 42", phone: "062-220-5114", website: "https://www.cnuh.com" },
    { name: "조선대학교병원", region: "광주·전남", address: "광주광역시 동구 필문대로 365", phone: "062-220-3000", website: "https://www.csch.or.kr" },
    { name: "화순전남대학교병원", region: "광주·전남", address: "전라남도 화순군 화순읍 서양로 322", phone: "061-379-7114", website: "https://www.hcc.ac.kr" },

    // ——— 경북권 (5) ———
    { name: "경북대학교병원", region: "대구·경북", address: "대구광역시 중구 동덕로 130", phone: "053-200-5114", website: "https://www.knuh.or.kr" },
    { name: "계명대학교 동산병원", region: "대구·경북", address: "대구광역시 달서구 달구벌대로 1035", phone: "053-258-7114", website: "https://www.dsmc.or.kr" },
    { name: "대구가톨릭대학교병원", region: "대구·경북", address: "대구광역시 남구 두류공원로 17", phone: "053-650-3000", website: "https://www.dcmc.co.kr" },
    { name: "영남대학교병원", region: "대구·경북", address: "대구광역시 남구 현충로 170", phone: "053-620-3114", website: "https://yumc.ac.kr" },
    { name: "칠곡경북대학교병원", region: "대구·경북", address: "대구광역시 북구 호국로 807", phone: "053-200-2000", website: "https://ck.knu.ac.kr" },

    // ——— 경남 동부권 (6) ———
    { name: "고신대학교 복음병원", region: "부산·울산·경남 동부", address: "부산광역시 서구 감천로 262", phone: "051-990-6114", website: "https://www.kosinmed.or.kr" },
    { name: "동아대학교병원", region: "부산·울산·경남 동부", address: "부산광역시 서구 대신공원로 26", phone: "051-240-2000", website: "https://www.damc.or.kr" },
    { name: "부산대학교병원", region: "부산·울산·경남 동부", address: "부산광역시 서구 구덕로 179", phone: "051-240-7000", website: "https://www.pnuh.or.kr" },
    { name: "양산부산대학교병원", region: "부산·울산·경남 동부", address: "경상남도 양산시 물금읍 금오로 20", phone: "055-360-1000", website: "https://www.ypuh.co.kr" },
    { name: "인제대학교 부산백병원", region: "부산·울산·경남 동부", address: "부산광역시 부산진구 복지로 75", phone: "051-890-6114", website: "https://busan.paik.ac.kr" },
    { name: "울산대학교병원", region: "부산·울산·경남 동부", address: "울산광역시 동구 산업로 877", phone: "052-250-7000", website: "https://www.uuh.ulsan.kr" },

    // ——— 경남 서부권 (2) ———
    { name: "경상국립대학교병원", region: "경남 서부", address: "경상남도 진주시 강남로 79", phone: "055-750-8711", website: "https://www.gnuh.co.kr" },
    { name: "학교법인 성균관대학 삼성창원병원", region: "경남 서부", address: "경상남도 창원시 마산회원구 팔용로 158", phone: "055-233-2000", website: "https://www.smc.or.kr" },
  ];

  // ===== 상태 =====
  let hospitals = JSON.parse(JSON.stringify(initialHospitals));
  let regionFilter = '전체';

  // ===== 유틸 =====
  const $ = (sel) => document.querySelector(sel);
  const $$ = (sel) => Array.from(document.querySelectorAll(sel));

  function normalizePhoneForTel(phone){
    if(!phone) return '';
    const digits = phone.replace(/[^0-9+]/g,'');
    return digits.startsWith('+') ? digits : digits;
  }
  async function copyText(text){
    try{ await navigator.clipboard.writeText(text); toast('복사했습니다'); }catch(e){ alert('복사 실패: 권한을 확인하세요'); }
  }
  function toast(msg){
    const el = document.createElement('div');
    el.textContent = msg; el.className = 'fixed bottom-6 left-1/2 -translate-x-1/2 z-50 rounded-full px-4 py-2 bg-black/80 text-white text-sm shadow-lg';
    document.body.appendChild(el);
    setTimeout(()=>{ el.classList.add('opacity-0'); setTimeout(()=>el.remove(),250); }, 1200);
  }
  function openNaverMap(h){
    const q = encodeURIComponent(`${h.name} ${h.address}`.trim());
    window.open(`https://map.naver.com/v5/search/${q}`,'_blank');
  }

  // ===== 렌더링 =====
  function renderRegions(){
    const wrap = $('#regionWrap');
    const regions = ['전체', ...Array.from(new Set(hospitals.map(h=>h.region)))];
    wrap.innerHTML = regions.map(r=>`
      <button data-region="${r}" class="btn rounded-full px-4 py-2 text-sm border ${regionFilter===r? 'bg-sky-600 text-white border-sky-600' : 'border-slate-200 dark:border-slate-700 hover:bg-slate-50 dark:hover:bg-slate-800'}">${r}</button>
    `).join('');
    wrap.querySelectorAll('button').forEach(btn=>{
      btn.addEventListener('click',()=>{ regionFilter = btn.dataset.region; renderList(); renderRegions(); });
    });
  }

  function cardTpl(h){
    const tel = h.phone? `tel:${normalizePhoneForTel(h.phone)}` : '';
    return `
      <article class="card rounded-2xl border border-slate-200 dark:border-slate-800 bg-white/80 dark:bg-slate-900/70">
        <div class="p-5 flex flex-col h-full">
          <header class="mb-3">
            <h3 class="text-lg font-bold leading-snug">${h.name}</h3>
            <p class="text-xs mt-1 inline-flex items-center gap-1 text-slate-500 dark:text-slate-400">
              <i data-lucide="map-pin" class="w-4 h-4"></i>
              <span>${h.region}</span>
            </p>
          </header>
          <div class="text-sm flex-1">
            <div class="mb-2">
              <p class="text-slate-600 dark:text-slate-300 leading-relaxed">${h.address||'주소 준비 중'}</p>
              <div class="mt-1 flex gap-2 flex-wrap">
                ${h.address? `<button data-action="copyAddress" class="btn inline-flex items-center gap-1 text-xs px-2 py-1 rounded-full border border-slate-200 dark:border-slate-700 hover:bg-slate-50 dark:hover:bg-slate-800"><i data-lucide="copy" class="w-3.5 h-3.5"></i> 주소 복사</button>`:''}
                <button data-action="naverMap" class="btn inline-flex items-center gap-1 text-xs px-2 py-1 rounded-full border border-slate-200 dark:border-slate-700 hover:bg-slate-50 dark:hover:bg-slate-800"><i data-lucide="globe" class="w-3.5 h-3.5"></i> 네이버지도</button>
              </div>
            </div>
            <div class="mt-3">
              <span class="text-slate-500 dark:text-slate-400">대표번호</span>
              <div class="mt-1 flex items-center gap-2 flex-wrap">
                <a ${tel? `href="${tel}"`:''} class="btn inline-flex items-center gap-2 rounded-full px-3 py-2 text-sm ${tel? 'bg-emerald-600 text-white hover:bg-emerald-700':'bg-slate-200 text-slate-500 cursor-not-allowed'}">
                  <i data-lucide="phone" class="w-4 h-4"></i> ${h.phone||'미등록'}
                </a>
                ${h.phone? `<button data-action="copyPhone" class="btn inline-flex items-center gap-1 text-xs px-2 py-1 rounded-full border border-slate-200 dark:border-slate-700 hover:bg-slate-50 dark:hover:bg-slate-800"><i data-lucide="copy" class="w-3.5 h-3.5"></i> 복사</button>`:''}
              </div>
            </div>
          </div>
          ${h.website? `<footer class="mt-4 pt-4 border-t border-slate-200 dark:border-slate-800"><a href="${h.website}" target="_blank" rel="noreferrer" class="text-sm text-sky-700 dark:text-sky-300 underline hover:no-underline">공식 홈페이지 방문</a></footer>`:''}
        </div>
      </article>
    `;
  }

  function renderList(){
    const q = ($('#searchInput').value||'').trim().toLowerCase();
    const arr = hospitals.filter(h=>{
      const hitRegion = regionFilter==='전체' || h.region===regionFilter;
      const hitQuery = !q || [h.name,h.address,h.region,h.phone||''].some(v=> (v||'').toLowerCase().includes(q));
      return hitRegion && hitQuery;
    });

    const list = $('#list');
    if(arr.length===0){ $('#empty').classList.remove('hidden'); list.innerHTML=''; lucide.createIcons(); return; }
    $('#empty').classList.add('hidden');
    list.innerHTML = arr.map(cardTpl).join('');

    // 이벤트 바인딩
    list.querySelectorAll('article').forEach((card, idx)=>{
      const h = arr[idx];
      const copyAddrBtn = card.querySelector('[data-action="copyAddress"]');
      if(copyAddrBtn) copyAddrBtn.addEventListener('click', ()=>copyText(h.address));

      const mapBtn = card.querySelector('[data-action="naverMap"]');
      mapBtn.addEventListener('click', ()=>openNaverMap(h));

      const copyPhoneBtn = card.querySelector('[data-action="copyPhone"]');
      if(copyPhoneBtn) copyPhoneBtn.addEventListener('click', ()=>copyText(h.phone));
    });

    // 아이콘 렌더
    lucide.createIcons();
  }

  // ===== CSV 처리 =====
  function downloadCSV(){
    const header = ['name','region','address','phone','website'];
    const rows = hospitals.map(h => header.map(k => (h[k]||'').replaceAll('"','""')));
    const csv = [header.join(','), ...rows.map(r=> r.map(v=> /[",\n]/.test(v)? `"${v}"`: v).join(','))].join('\n');
    const blob = new Blob([csv],{type:'text/csv;charset=utf-8;'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a'); a.href = url; a.download = 'hospitals.csv'; a.click(); URL.revokeObjectURL(url);
  }
  function parseCSV(text){
    const lines = text.split(/\r?\n/).filter(Boolean);
    if(!lines.length) return [];
    const header = lines[0].split(',').map(s=>s.trim());
    const out = [];
    for(let i=1;i<lines.length;i++){
      const row=[]; let cur=''; let inQ=false; const line=lines[i];
      for(let j=0;j<line.length;j++){
        const ch=line[j];
        if(ch==='"') { if(inQ && line[j+1]==='"'){cur+='"'; j++;} else inQ=!inQ; }
        else if(ch===',' && !inQ){ row.push(cur); cur=''; }
        else { cur+=ch; }
      }
      row.push(cur);
      const obj={}; header.forEach((k,idx)=> obj[k]=(row[idx]||'').trim());
      out.push(obj);
    }
    return out;
  }

  // ===== 초기화 =====
  document.addEventListener('DOMContentLoaded', ()=>{
    // 다크모드 초기
    const saved = localStorage.getItem('dark');
    if(saved==='1') document.documentElement.classList.add('dark');

    // 이벤트
    $('#modeBtn').addEventListener('click', ()=>{
      document.documentElement.classList.toggle('dark');
      const on = document.documentElement.classList.contains('dark');
      localStorage.setItem('dark', on?'1':'0');
      lucide.createIcons();
    });

    $('#csvDownBtn').addEventListener('click', downloadCSV);
    $('#csvUpBtn').addEventListener('click', ()=>{ $('#importModal').classList.remove('hidden'); $('#importModal').classList.add('flex'); });
    $('#modalCloseBtn').addEventListener('click', ()=>{ $('#importModal').classList.add('hidden'); $('#importModal').classList.remove('flex'); });
    $('#modalImportBtn').addEventListener('click', ()=>{
      const text = $('#csvTextarea').value||'';
      if(!text.trim()) return;
      try{
        const arr = parseCSV(text);
        if(!arr.length) return alert('CSV를 해석할 수 없습니다. 첫 줄에 헤더가 필요합니다.');
        const normalized = arr.map(h=>({
          name: h.name||'', region: h.region||'', address: h.address||'', phone: h.phone||'', website: h.website||''
        })).filter(h=>h.name && h.address);
        hospitals = normalized; regionFilter='전체'; $('#searchInput').value='';
        renderRegions(); renderList();
        $('#importModal').classList.add('hidden'); $('#importModal').classList.remove('flex');
        toast('데이터가 업데이트되었습니다');
      }catch(e){ alert('가져오기 실패: CSV 형식을 확인하세요.'); }
    });

    $('#shareBtn').addEventListener('click', async ()=>{
      try{
        const data = { title:'상급종합병원 디렉터리', text:'상급종합병원(제5기) 47개 연락처·주소 모음', url: location.href };
        if(navigator.share){ await navigator.share(data); }
        else { await navigator.clipboard.writeText(location.href); toast('현재 페이지 주소를 복사했어요'); }
      }catch(e){ await navigator.clipboard.writeText(location.href); toast('현재 페이지 주소를 복사했어요'); }
    });

    $('#resetBtn').addEventListener('click', ()=>{ hospitals = JSON.parse(JSON.stringify(initialHospitals)); regionFilter='전체'; $('#searchInput').value=''; renderRegions(); renderList(); toast('초기 데이터로 복원했습니다'); });

    $('#searchInput').addEventListener('input', ()=> renderList());

    // 첫 렌더
    renderRegions();
    renderList();
    lucide.createIcons();
  });
  </script>
</body>
</html>
