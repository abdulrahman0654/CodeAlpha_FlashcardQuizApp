# CodeAlpha_FlashcardQuizA
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Flashcard Quiz App</title>
<style>
  /* Simple, clean, mobile friendly styles */
  :root{--bg:#f7fafc;--card:#ffffff;--accent:#3b82f6;--muted:#6b7280}
  body{margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial;background:var(--bg);color:#111}
  .app{max-width:720px;margin:18px auto;padding:16px}
  header{display:flex;justify-content:space-between;align-items:center;margin-bottom:12px}
  h1{font-size:20px;margin:0}
  .card{background:var(--card);border-radius:12px;padding:18px;box-shadow:0 6px 18px rgba(2,6,23,0.06)}
  .question{font-size:18px;margin:0 0 8px}
  .answer{font-size:16px;color:var(--muted);margin:8px 0;display:none}
  .show .answer{display:block}
  .controls{display:flex;gap:8px;margin-top:12px;flex-wrap:wrap}
  button{background:var(--accent);color:white;border:none;padding:10px 12px;border-radius:10px;font-size:14px}
  button.ghost{background:transparent;color:var(--accent);border:1px solid rgba(59,130,246,0.18)}
  button.danger{background:#ef4444}
  .small{padding:8px 10px;font-size:13px;border-radius:8px}
  .nav{display:flex;gap:8px;margin-top:12px;justify-content:center}
  .meta{margin-top:10px;color:var(--muted);font-size:13px}
  .form-row{display:flex;gap:8px;margin-top:10px}
  input[type="text"], textarea{flex:1;padding:10px;border-radius:10px;border:1px solid #e6edf6;font-size:14px}
  .list{margin-top:14px}
  .list-item{display:flex;align-items:center;justify-content:space-between;padding:8px 0;border-bottom:1px dashed #eaeef6}
  .list-left{font-size:14px}
  .btn-inline{background:transparent;border:none;color:var(--accent);padding:6px;font-size:13px}
  .center{display:flex;justify-content:center}
  @media (max-width:420px){.app{padding:12px} h1{font-size:18px} .question{font-size:16px}}
</style>
</head>
<body>
<div class="app">
  <header>
    <h1>Flashcard Quiz App</h1>
    <div>
      <button id="addBtn" class="small">+ Add</button>
    </div>
  </header>

  <div class="card" id="cardContainer">
    <p class="question" id="questionText">No flashcards yet — add one.</p>
    <p class="answer" id="answerText"></p>

    <div class="controls">
      <button id="showBtn" class="ghost small">Show Answer</button>
      <div style="flex:1"></div>
      <button id="editBtn" class="btn-inline">Edit</button>
      <button id="deleteBtn" class="btn-inline" style="color:#ef4444">Delete</button>
    </div>

    <div class="nav">
      <button id="prevBtn" class="small">Previous</button>
      <div style="width:10px"></div>
      <button id="nextBtn" class="small">Next</button>
    </div>

    <div class="meta" id="metaInfo">0 / 0</div>
  </div>

  <div class="card" style="margin-top:12px">
    <strong>Create / Edit flashcard</strong>
    <div class="form-row">
      <input id="qInput" type="text" placeholder="Question (front)">
    </div>
    <div class="form-row" style="margin-top:8px">
      <textarea id="aInput" rows="3" placeholder="Answer (back)"></textarea>
    </div>
    <div style="display:flex;gap:8px;margin-top:8px">
      <button id="saveBtn">Save</button>
      <button id="clearBtn" class="ghost">Clear</button>
    </div>
    <div class="meta">Note: Cards are stored on your device (localStorage).</div>
  </div>

  <div class="card list" id="listContainer" style="margin-top:12px">
    <strong>Your flashcards</strong>
    <div id="list"></div>
  </div>

  <div style="height:36px"></div>
</div>

<script>
/* Simple flashcard app using localStorage */
const STORAGE_KEY = 'flashcards_v1'
let flashcards = []
let current = 0
let editingId = null
const qText = document.getElementById('questionText')
const aText = document.getElementById('answerText')
const meta = document.getElementById('metaInfo')
const showBtn = document.getElementById('showBtn')
const nextBtn = document.getElementById('nextBtn')
const prevBtn = document.getElementById('prevBtn')
const addBtn = document.getElementById('addBtn')
const saveBtn = document.getElementById('saveBtn')
const clearBtn = document.getElementById('clearBtn')
const qInput = document.getElementById('qInput')
const aInput = document.getElementById('aInput')
const editBtn = document.getElementById('editBtn')
const deleteBtn = document.getElementById('deleteBtn')
const listDiv = document.getElementById('list')
const cardContainer = document.getElementById('cardContainer')

/* load */
function load() {
  const raw = localStorage.getItem(STORAGE_KEY)
  flashcards = raw ? JSON.parse(raw) : []
  if (flashcards.length === 0) {
    // add a sample card for first time
    flashcards.push({id: Date.now(), question: "Tap +Add to create a card", answer: "Use Next/Previous to navigate."})
  }
  saveStorage()
  renderList()
  showCard(0)
}

/* save */
function saveStorage() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(flashcards))
}

/* render list */
function renderList() {
  listDiv.innerHTML = ''
  flashcards.forEach((c, i) => {
    const el = document.createElement('div')
    el.className = 'list-item'
    el.innerHTML = `<div class="list-left">${i+1}. ${escapeHtml(c.question)}</div>
      <div>
        <button class="btn-inline" data-i="${i}" onclick="jumpTo(event)">Open</button>
        <button class="btn-inline" data-i="${i}" onclick="startEdit(event)">Edit</button>
        <button class="btn-inline" data-i="${i}" onclick="doDelete(event)" style="color:#ef4444">Delete</button>
      </div>`
    listDiv.appendChild(el)
  })
  meta.innerText = \`\${flashcards.length ? current+1 : 0} / \${flashcards.length}\`
}

/* display card */
function showCard(i) {
  if (flashcards.length === 0) {
    qText.innerText = 'No flashcards yet — add one.'
    aText.innerText = ''
    cardContainer.classList.remove('show')
    meta.innerText = '0 / 0'
    return
  }
  if (i < 0) i = 0
  if (i >= flashcards.length) i = flashcards.length - 1
  current = i
  qText.innerText = flashcards[current].question
  aText.innerText = flashcards[current].answer
  cardContainer.classList.remove('show')
  showBtn.innerText = 'Show Answer'
  meta.innerText = \`\${current+1} / \${flashcards.length}\`
}

/* next/prev */
nextBtn.onclick = ()=> showCard((current + 1) % Math.max(1, flashcards.length))
prevBtn.onclick = ()=> showCard((current - 1 + flashcards.length) % Math.max(1, flashcards.length))

/* show/hide answer */
showBtn.onclick = ()=>{
  if (cardContainer.classList.contains('show')) {
    cardContainer.classList.remove('show')
    showBtn.innerText = 'Show Answer'
  } else {
    cardContainer.classList.add('show')
    showBtn.innerText = 'Hide Answer'
  }
}

/* add new - prepare form */
addBtn.onclick = ()=>{
  editingId = null
  qInput.value = ''
  aInput.value = ''
  qInput.focus()
}

/* save from form (add or update) */
saveBtn.onclick = ()=>{
  const q = qInput.value.trim()
  const a = aInput.value.trim()
  if (!q) { alert('Add a question first'); qInput.focus(); return }
  if (editingId) {
    // update
    const idx = flashcards.findIndex(c=>c.id===editingId)
    if (idx >= 0) {
      flashcards[idx].question = q
      flashcards[idx].answer = a
      saveStorage()
      renderList()
      showCard(idx)
      editingId = null
      qInput.value=''; aInput.value=''
      return
    }
  }
  // add new
  const card = { id: Date.now(), question: q, answer: a }
  flashcards.push(card)
  saveStorage()
  renderList()
  showCard(flashcards.length - 1)
  qInput.value=''; aInput.value=''
}

/* clear form */
clearBtn.onclick = ()=>{
  editingId = null
  qInput.value = ''
  aInput.value = ''
}

/* edit from top-right edit button */
editBtn.onclick = ()=>{
  if (!flashcards.length) return
  const c = flashcards[current]
  editingId = c.id
  qInput.value = c.question
  aInput.value = c.answer
  qInput.focus()
}

/* delete current */
deleteBtn.onclick = ()=>{
  if (!flashcards.length) return
  if (!confirm('Delete this flashcard?')) return
  flashcards.splice(current,1)
  saveStorage()
  renderList()
  showCard(Math.min(current, flashcards.length - 1))
}

/* helpers used in list */
function jumpTo(e){
  const i = Number(e.target.getAttribute('data-i'))
  showCard(i)
}
function startEdit(e){
  const i = Number(e.target.getAttribute('data-i'))
  const c = flashcards[i]
  editingId = c.id
  qInput.value = c.question
  aInput.value = c.answer
  window.scrollTo({top:0,behavior:'smooth'})
}
function doDelete(e){
  const i = Number(e.target.getAttribute('data-i'))
  if (!confirm('Delete this flashcard?')) return
  flashcards.splice(i,1)
  saveStorage()
  renderList()
  showCard(Math.min(current, flashcards.length - 1))
}

function escapeHtml(text){ return text.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;') }

/* initial load */
load()

/* make functions available for inline onclick from created elements */
window.jumpTo = jumpTo
window.startEdit = startEdit
window.doDelete = doDelete
</script>
</body>
</html>
