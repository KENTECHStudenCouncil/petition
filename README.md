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
<main id="page-main" class="container mx-auto p-6">
  <h2 class="text-4xl font-bold mb-6">🔥 HOT 청원</h2>
  <div id="hot-petitions" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 mb-10">
    <!-- HOT 청원 카드 동적 생성 -->
  </div>

  <h2 class="text-3xl font-bold mb-4">📜 최근 청원</h2>
  <ul id="recent-petitions" class="divide-y divide-gray-300">
    <!-- 최근 청원 동적 생성 -->
  </ul>
</main>
<section id="page-list" class="hidden container mx-auto p-6">
  <h2 class="text-3xl font-bold mb-6">전체 청원 목록</h2>
  <ul id="all-petitions" class="divide-y divide-gray-300">
    <!-- 전체 청원 동적 생성 -->
  </ul>
</section>
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

  <div class="bg-gray-50 p-4 rounded">
    <h3 class="text-xl font-semibold mb-4">댓글 (실명제)</h3>
    <input type="text" class="w-full border p-2 mb-2 rounded" placeholder="댓글 작성 (이름)">
    <button class="bg-blue-600 text-white px-4 py-2 rounded">댓글 작성</button>
  </div>
</section>
<section id="page-write" class="hidden container mx-auto p-6">
  <h2 class="text-3xl font-bold mb-6">청원 작성하기</h2>
  <input id="petition-title" type="text" class="w-full border p-2 mb-4 rounded" placeholder="청원 제목">
  <textarea id="petition-content" class="w-full border p-2 mb-4 rounded" placeholder="청원 내용"></textarea>
  <button onclick="submitPetition()" class="bg-blue-700 text-white px-6 py-2 rounded">제출하기</button>
</section>
<section id="page-mypage" class="hidden container mx-auto p-6">
  <h2 class="text-3xl font-bold mb-6">마이페이지</h2>

  <h3 class="text-2xl font-semibold mb-4">내 청원</h3>
  <ul id="my-petitions" class="divide-y divide-gray-300 mb-6">
    <!-- 내가 작성한 청원 불러오기 -->
  </ul>

  <h3 class="text-2xl font-semibold mb-4">내 서명</h3>
  <ul id="my-supports" class="divide-y divide-gray-300 mb-6">
    <!-- 내가 서명한 청원 불러오기 -->
  </ul>

  <h3 class="text-2xl font-semibold mb-4">내 댓글</h3>
  <ul id="my-comments" class="divide-y divide-gray-300">
    <!-- 내가 쓴 댓글 불러오기 (추후 구현) -->
  </ul>
</section>
<section id="page-admin" class="hidden container mx-auto p-6">
  <h2 class="text-3xl font-bold mb-6">관리자 페이지 (준비중)</h2>
</section>
<script>

function showPage(page) {
    const pages = ['main', 'list', 'detail', 'write', 'mypage', 'admin'];
    pages.forEach(id => {
      const section = document.getElementById(`page-${id}`);
      if (section) section.classList.add('hidden');
    });
    const show = document.getElementById(`page-${page}`);
    if (show) show.classList.remove('hidden');
  }

// Supabase 연결
const supabaseUrl = 'https://ybbpzwvigqgleywnwkij.supabase.co'; // 본인 프로젝트 URL
const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InliYnB6d3ZpZ3FnbGV5d253a2lqIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDU5Mjk1NzUsImV4cCI6MjA2MTUwNTU3NX0.3JF0NvkBLyJZkFtcpOvtYkA8CfUnp_CKuAoI13CyJxg';               // 본인 Public Key
const supabase = supabase.createClient(supabaseUrl, supabaseKey);

let currentPetition = null;

// 페이지 전환
function showPage(page) {
  const pages = ['main', 'list', 'detail', 'write', 'mypage', 'admin'];
  pages.forEach(id => {
    document.getElementById(`page-${id}`)?.classList.add('hidden');
  });
  document.getElementById(`page-${page}`)?.classList.remove('hidden');
}
async function submitPetition() {
  const title = document.getElementById('petition-title').value;
  const content = document.getElementById('petition-content').value;

  if (!title || !content) {
    alert('모든 칸을 입력해 주세요.');
    return;
  }

  const { data, error } = await supabase
    .from('petitions')
    .insert([{ title: title, description: content, support_count: 0 }]);

  if (error) {
    console.error(error.message);
    alert('청원 등록 실패');
  } else {
    alert('청원 등록 성공!');
    showPage('main');
    loadRecentPetitions();
    loadAllPetitions();
  }
}
async function loadRecentPetitions() {
  const { data, error } = await supabase
    .from('petitions')
    .select('*')
    .order('created_at', { ascending: false })
    .limit(10);

  const list = document.getElementById('recent-petitions');
  list.innerHTML = '';

  data.forEach(petition => {
    const li = document.createElement('li');
    li.className = 'py-2 flex justify-between cursor-pointer hover:text-blue-600';
    li.innerHTML = `<span>${petition.title}</span><span class="text-gray-500">${new Date(petition.created_at).toLocaleDateString()}</span>`;
    li.onclick = () => openDetail(petition);
    list.appendChild(li);
  });
}

async function loadAllPetitions() {
  const { data, error } = await supabase
    .from('petitions')
    .select('*')
    .order('created_at', { ascending: false });

  const list = document.getElementById('all-petitions');
  list.innerHTML = '';

  data.forEach(petition => {
    const li = document.createElement('li');
    li.className = 'py-4 flex justify-between cursor-pointer hover:text-blue-600';
    li.innerHTML = `<span>${petition.title}</span><span>동의 ${petition.support_count}명</span>`;
    li.onclick = () => openDetail(petition);
    list.appendChild(li);
  });
}
function openDetail(petition) {
  currentPetition = petition;

  document.getElementById('detail-title').textContent = petition.title;
  document.getElementById('detail-description').textContent = petition.description;
  document.getElementById('detail-support').textContent = `동의 ${petition.support_count}명`;

  document.getElementById('support-name').value = '';
  document.getElementById('support-file').value = '';

  showPage('detail');
}
async function submitSupport() {
  const name = document.getElementById('support-name').value;
  const file = document.getElementById('support-file').files[0];

  if (!name) {
    alert('이름을 입력해야 합니다.');
    return;
  }
  if (!file) {
    alert('서명 파일을 업로드해야 합니다.');
    return;
  }

  // 파일 업로드
  const fileName = `${Date.now()}_${file.name}`;
  const { data: uploadData, error: uploadError } = await supabase
    .storage
    .from('signatures')
    .upload(fileName, file);

  if (uploadError) {
    console.error(uploadError.message);
    alert('파일 업로드 실패');
    return;
  }

  const fileUrl = `${supabaseUrl}/storage/v1/object/public/signatures/${fileName}`;

  // supports 테이블에 저장
  await supabase
    .from('supports')
    .insert([{
      petition_id: currentPetition.id,
      name: name,
      file_url: fileUrl
    }]);

  // support_count +1 업데이트
  const updatedCount = currentPetition.support_count + 1;
  await supabase
    .from('petitions')
    .update({ support_count: updatedCount })
    .eq('id', currentPetition.id);

  alert('서명 완료!');
  
  loadRecentPetitions();
  loadAllPetitions();
  showPage('main');
}
window.onload = function() {
  loadRecentPetitions();
  loadAllPetitions();
};
<footer class="bg-gray-800 text-white text-center p-4 mt-12">
  © 2025 Kentech Petitions
</footer>
