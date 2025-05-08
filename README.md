<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Kentech Petitions</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js"></script>
  <style>body { font-family: 'Noto Sans KR', sans-serif; }</style>
</head>
<body class="bg-gray-100 text-gray-900">

<!-- 상단 네비게이션 -->
<header class="bg-blue-700 text-white p-4 flex justify-between items-center">
  <h1 class="text-2xl font-bold">Kentech Petitions</h1>
  <nav class="space-x-4">
    <button onclick="showPage('main')">홈</button>
    <button onclick="showPage('list')">전체 청원</button>
    <button onclick="showPage('write')">청원 작성</button>
    <button onclick="showPage('mypage')">마이페이지</button>
    <button onclick="showPage('admin')">관리자</button>
  </nav>
</header>

<!-- 메인 페이지 -->
<main id="page-main" class="container mx-auto p-6">
  <h2 class="text-4xl font-bold mb-6">🔥 HOT 청원</h2>
  <div id="hot-petitions" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 mb-10"></div>

  <h2 class="text-3xl font-bold mb-4">📜 최근 청원</h2>
  <ul id="recent-petitions" class="divide-y divide-gray-300"></ul>
</main>

<!-- 전체 청원 목록 -->
<section id="page-list" class="hidden container mx-auto p-6">
  <h2 class="text-3xl font-bold mb-6">전체 청원 목록</h2>
  <ul id="all-petitions" class="divide-y divide-gray-300"></ul>
</section>

<!-- 청원 상세 페이지 -->
<section id="page-detail" class="hidden container mx-auto p-6">
  <h2 id="detail-title" class="text-3xl font-bold mb-4">청원 제목</h2>
  <p id="detail-description" class="text-gray-700 mb-6">청원 내용</p>
  <p id="detail-support" class="text-green-600 font-semibold mb-6">동의 0명</p>

  <div class="bg-gray-50 p-4 rounded mb-6">
    <h3 class="text-xl font-semibold mb-4">서명하기</h3>
    <input id="support-name" type="text" class="w-full border p-2 mb-2 rounded" placeholder="이름 (필수)">
    <input id="support-file" type="file" class="w-full mb-2">
    <button onclick="submitSupport()" class="bg-green-600 text-white px-4 py-2 rounded">동의하기</button>
  </div>
</section>

<!-- 청원 작성 페이지 -->
<section id="page-write" class="hidden container mx-auto p-6">
  <h2 class="text-3xl font-bold mb-6">청원 작성하기</h2>
  <input id="petition-title" type="text" class="w-full border p-2 mb-4 rounded" placeholder="청원 제목">
  <textarea id="petition-content" class="w-full border p-2 mb-4 rounded" placeholder="청원 내용"></textarea>
  <button onclick="submitPetition()" class="bg-blue-700 text-white px-6 py-2 rounded">제출하기</button>
</section>

<!-- 마이페이지 & 관리자 페이지 생략 (추후 구현) -->

<!-- 스크립트 시작 -->
<script>
const supabaseUrl = 'https://your-project-id.supabase.co](https://ybbpzwvigqgleywnwkij.supabase.co'; // ← 본인의 URL로 교체
const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InliYnB6d3ZpZ3FnbGV5d253a2lqIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDU5Mjk1NzUsImV4cCI6MjA2MTUwNTU3NX0.3JF0NvkBLyJZkFtcpOvtYkA8CfUnp_CKuAoI13CyJxg'; // ← 본인의 Public Key로 교체
const supabase = supabase.createClient(supabaseUrl, supabaseKey);

let currentPetition = null;

function showPage(page) {
  const pages = ['main', 'list', 'detail', 'write'];
  pages.forEach(id => {
    const el = document.getElementById(`page-${id}`);
    if (el) el.classList.add('hidden');
  });
  const showEl = document.getElementById(`page-${page}`);
  if (showEl) showEl.classList.remove('hidden');
}

async function submitPetition() {
  const title = document.getElementById('petition-title').value;
  const content = document.getElementById('petition-content').value;
  if (!title || !content) return alert('모든 항목을 입력해주세요.');

  const { error } = await supabase.from('petitions').insert([{ title, description: content, support_count: 0 }]);
  if (error) return alert('청원 등록 실패: ' + error.message);

  alert('청원이 등록되었습니다.');
  showPage('main');
  loadRecentPetitions();
  loadAllPetitions();
}

async function loadRecentPetitions() {
  const { data } = await supabase
    .from('petitions')
    .select('*')
    .order('created_at', { ascending: false })
    .limit(10);

  const list = document.getElementById('recent-petitions');
  list.innerHTML = '';
  data?.forEach(p => {
    const li = document.createElement('li');
    li.className = 'py-2 flex justify-between cursor-pointer hover:text-blue-600';
    li.innerHTML = `<span>${p.title}</span><span class="text-gray-500">${new Date(p.created_at).toLocaleDateString()}</span>`;
    li.onclick = () => openDetail(p);
    list.appendChild(li);
  });
}

async function loadAllPetitions() {
  const { data } = await supabase.from('petitions').select('*').order('created_at', { ascending: false });
  const list = document.getElementById('all-petitions');
  list.innerHTML = '';
  data?.forEach(p => {
    const li = document.createElement('li');
    li.className = 'py-4 flex justify-between cursor-pointer hover:text-blue-600';
    li.innerHTML = `<span>${p.title}</span><span>동의 ${p.support_count}명</span>`;
    li.onclick = () => openDetail(p);
    list.appendChild(li);
  });
}

function openDetail(petition) {
  currentPetition = petition;
  document.getElementById('detail-title').textContent = petition.title;
  document.getElementById('detail-description').textContent = petition.description;
  document.getElementById('detail-support').textContent = `동의 ${petition.support_count}명`;
  showPage('detail');
}

async function submitSupport() {
  const name = document.getElementById('support-name').value;
  const file = document.getElementById('support-file').files[0];
  if (!name || !file) return alert('이름과 서명 파일을 모두 제출해주세요.');

  const filename = `${Date.now()}_${file.name}`;
  const { error: uploadError } = await supabase.storage.from('signatures').upload(filename, file);
  if (uploadError) return alert('파일 업로드 실패: ' + uploadError.message);

  const fileUrl = `${supabaseUrl}/storage/v1/object/public/signatures/${filename}`;

  const { error } = await supabase.from('supports').insert([{ petition_id: currentPetition.id, name, file_url: fileUrl }]);
  if (error) return alert('서명 실패: ' + error.message);

  await supabase
    .from('petitions')
    .update({ support_count: currentPetition.support_count + 1 })
    .eq('id', currentPetition.id);

  alert('서명 완료!');
  showPage('main');
  loadRecentPetitions();
  loadAllPetitions();
}

window.onload = () => {
  loadRecentPetitions();
  loadAllPetitions();
};
</script>

<footer class="bg-gray-800 text-white text-center p-4 mt-12">© 2025 Kentech Petitions</footer>
</body>
</html>
