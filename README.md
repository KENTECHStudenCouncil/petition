<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Kentech Petitions</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@1"></script>
  <style>body { font-family: 'Noto Sans KR', sans-serif; }</style>
</head>
<body class="bg-gray-100 text-gray-900">

<!-- 이하 기존 내용 동일 -->

<script>
let currentPetition = null;
let supabase = null;

window.onload = async () => {
  const supabaseUrl = 'https://ybbpzwvigqgleywnwkij.supabase.co';
  const supabaseKey = 'YOUR_SUPABASE_PUBLIC_KEY'; // 여기에 Supabase anon key 입력
  supabase = supabase.createClient(supabaseUrl, supabaseKey);

  await loadRecentPetitions();
  await loadAllPetitions();
  await loadHotPetitions();
  await loadUnapprovedPetitions();
};

function showPage(page) {
  const pages = ['main', 'list', 'detail', 'write', 'admin'];
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

  const { error } = await supabase.from('petitions').insert([
    { title, description: content, support_count: 0, approved: false }
  ]);

  if (error) return alert('청원 등록 실패: ' + error.message);
  alert('청원이 등록되었습니다. 관리자의 승인을 기다립니다.');
  showPage('main');
  await loadRecentPetitions();
  await loadAllPetitions();
  await loadHotPetitions();
}

async function loadRecentPetitions() {
  const { data } = await supabase.from('petitions')
    .select('*')
    .eq('approved', true)
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
  const { data } = await supabase.from('petitions')
    .select('*')
    .eq('approved', true)
    .order('created_at', { ascending: false });
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

async function loadHotPetitions() {
  const { data } = await supabase.from('petitions')
    .select('*')
    .eq('approved', true)
    .order('support_count', { ascending: false })
    .limit(3);
  const container = document.getElementById('hot-petitions');
  container.innerHTML = '';
  data?.forEach(p => {
    const div = document.createElement('div');
    div.className = 'bg-white p-4 rounded shadow cursor-pointer hover:bg-blue-50';
    div.innerHTML = `<h3 class="text-xl font-bold mb-2">${p.title}</h3><p class="text-gray-600">동의 ${p.support_count}명</p>`;
    div.onclick = () => openDetail(p);
    container.appendChild(div);
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

  const filename = `${Date.now()}_${encodeURIComponent(file.name)}`;
  const { error: uploadError } = await supabase.storage.from('signatures').upload(filename, file);
  if (uploadError) return alert('파일 업로드 실패: ' + uploadError.message);

  const fileUrl = `https://ybbpzwvigqgleywnwkij.supabase.co/storage/v1/object/public/signatures/${filename}`;
  const { error } = await supabase.from('supports').insert([
    { petition_id: currentPetition.id, name, file_url: fileUrl }
  ]);
  if (error) return alert('서명 실패: ' + error.message);

  await supabase
    .from('petitions')
    .update({ support_count: currentPetition.support_count + 1 })
    .eq('id', currentPetition.id);

  alert('서명 완료!');
  showPage('main');
  await loadRecentPetitions();
  await loadAllPetitions();
  await loadHotPetitions();
}

async function loadUnapprovedPetitions() {
  const { data } = await supabase.from('petitions')
    .select('*')
    .eq('approved', false)
    .order('created_at', { ascending: false });
  const list = document.getElementById('unapproved-petitions');
  list.innerHTML = '';
  data?.forEach(p => {
    const li = document.createElement('li');
    li.className = 'py-4 flex justify-between items-center';
    li.innerHTML = `<span>${p.title}</span><button class="bg-green-600 text-white px-4 py-1 rounded" onclick="approvePetition(${p.id})">승인</button>`;
    list.appendChild(li);
  });
}

async function approvePetition(id) {
  const { error } = await supabase.from('petitions').update({ approved: true }).eq('id', id);
  if (error) return alert('승인 실패: ' + error.message);
  alert('승인 완료!');
  await loadUnapprovedPetitions();
}
</script>

<footer class="bg-gray-800 text-white text-center p-4 mt-12">© 2025 Kentech Petitions</footer>
</body>
</html>
