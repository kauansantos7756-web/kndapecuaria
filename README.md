<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Gerenciador de Estoque - Mercado (PWA Lite)</title>
  <style>
    :root{--bg:#f6f7fb;--card:#ffffff;--accent:#0b72ff;--danger:#e55353;--muted:#6b7280;--success:#16a34a}
    *{box-sizing:border-box;font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial}
    body{margin:0;background:var(--bg);color:#111}
    header{background:linear-gradient(90deg,#0b72ff,#4aa3ff);color:white;padding:18px 20px;display:flex;align-items:center;justify-content:space-between}
    header h1{margin:0;font-size:18px}
    header .sub{font-size:12px;opacity:.95}
    .container{max-width:1100px;margin:18px auto;padding:0 16px}
    .grid{display:grid;grid-template-columns:1fr 360px;gap:18px}
    .card{background:var(--card);border-radius:12px;box-shadow:0 6px 18px rgba(18,24,40,.06);padding:14px}
    .controls{display:flex;gap:8px;align-items:center}
    .btn{background:var(--accent);color:white;padding:8px 12px;border-radius:8px;border:none;cursor:pointer}
    .btn.ghost{background:transparent;color:var(--accent);border:1px solid rgba(11,114,255,.15)}
    .btn.danger{background:var(--danger)}
    input,select{padding:8px 10px;border-radius:8px;border:1px solid #e6e9ef}
    table{width:100%;border-collapse:collapse}
    th,td{padding:8px;text-align:left;border-bottom:1px solid #f1f3f6;font-size:14px}
    th{font-weight:600;color:var(--muted)}
    .small{font-size:13px;color:var(--muted)}
    .kpi{display:flex;gap:10px;align-items:center}
    .kpi div{flex:1}
    .badge{padding:6px 8px;border-radius:999px;background:#f1f7ff;color:var(--accent);font-weight:600}
    .list-scroll{max-height:480px;overflow:auto}
    .muted{color:var(--muted)}
    .danger-txt{color:var(--danger);font-weight:700}
    .success-txt{color:var(--success);font-weight:700}
    .flex{display:flex;gap:8px;align-items:center}
    .right{margin-left:auto}
    .search{flex:1}
    footer{max-width:1100px;margin:18px auto;padding:8px 16px;color:var(--muted);font-size:13px}
    /* modal */
    .modal-back{position:fixed;inset:0;background:rgba(2,6,23,.45);display:none;align-items:center;justify-content:center;padding:20px}
    .modal{background:var(--card);width:920px;max-width:100%;border-radius:10px;padding:16px}
    .row{display:flex;gap:10px}
    .col{flex:1}
    .tag{font-size:12px;padding:6px 8px;border-radius:6px;background:#f3f4f6}
    .chip{display:inline-block;padding:6px 8px;border-radius:8px;background:#f8fafc;font-weight:600}
    @media (max-width:880px){.grid{grid-template-columns:1fr}.card{padding:12px}.controls{flex-direction:column;align-items:stretch}.kpi{flex-direction:column}}
  </style>
</head>
<body>
  <header>
    <div>
      <h1>Gerenciador de Estoque — Mercado</h1>
      <div class="sub">App web para o dia a dia dos funcionários — cadastro, entradas, saídas, inventário e alertas.</div>
    </div>
    <div class="controls">
      <button class="btn" id="openAdd">+ Novo produto</button>
      <button class="btn ghost" id="exportCSV">Exportar CSV</button>
      <button class="btn ghost" id="importBtn">Importar CSV</button>
      <input type="file" id="importFile" accept="text/csv" style="display:none" />
    </div>
  </header>

  <main class="container">
    <div class="grid">
      <section class="card">
        <div style="display:flex;align-items:center;gap:12px;margin-bottom:12px">
          <div class="kpi" style="flex:1">
            <div>
              <div class="small">Produtos cadastrados</div>
              <div id="kCount" style="font-size:20px;font-weight:700">0</div>
            </div>
            <div>
              <div class="small">Produtos com estoque baixo</div>
              <div id="kLow" style="font-size:16px;color:#b45309;font-weight:700">0</div>
            </div>
          </div>
          <div style="min-width:220px;display:flex;gap:8px;align-items:center">
            <input class="search" id="search" placeholder="Buscar por nome, SKU ou EAN" />
            <select id="filterCat">
              <option value="">Todas categorias</option>
            </select>
          </div>
        </div>

        <div class="list-scroll">
          <table id="prodTable">
            <thead>
              <tr><th>Produto</th><th>SKU / EAN</th><th>Estoque</th><th>Validade</th><th class="right">Ações</th></tr>
            </thead>
            <tbody></tbody>
          </table>
        </div>

        <div style="margin-top:12px;display:flex;gap:8px;align-items:center">
          <button class="btn ghost" id="openInventory">Inventário rápido</button>
          <button class="btn ghost" id="resetDemo">Reset demo</button>
          <div class="muted" style="margin-left:auto">Dados salvos localmente (localStorage)</div>
        </div>
      </section>

      <aside class="card">
        <h3 style="margin:0 0 8px 0">Alertas e Ações Rápidas</h3>
        <div id="alerts"></div>
        <hr style="margin:12px 0" />
        <h4 style="margin:0 0 8px 0">Recebimento (entrada rápida)</h4>
        <div style="display:flex;gap:8px;margin-bottom:8px">
          <input id="rc_sku" placeholder="SKU ou EAN" />
          <input id="rc_qty" type="number" placeholder="Qtd" min="1" style="width:90px" />
          <button class="btn" id="rc_add">Adicionar</button>
        </div>
        <h4 style="margin:14px 0 8px 0">Saída rápida (venda/baixa)</h4>
        <div style="display:flex;gap:8px;margin-bottom:8px">
          <input id="out_sku" placeholder="SKU ou EAN" />
          <input id="out_qty" type="number" placeholder="Qtd" min="1" style="width:90px" />
          <button class="btn danger" id="out_sub">Registrar saída</button>
        </div>

        <hr style="margin:12px 0" />
        <h4 style="margin:0 0 8px 0">Relatórios rápidos</h4>
        <div style="display:flex;flex-direction:column;gap:8px">
          <button class="btn ghost" id="reportLow">Produtos com estoque baixo</button>
          <button class="btn ghost" id="reportExpiry">Próximos do vencimento (30d)</button>
          <button class="btn ghost" id="reportMovement">Movimentações (últimos 30d)</button>
        </div>
      </aside>
    </div>
  </main>

  <div class="modal-back" id="modalBack">
    <div class="modal" role="dialog" aria-modal="true">
      <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:8px">
        <h3 id="modalTitle">Novo Produto</h3>
        <div><button class="btn ghost" id="closeModal">Fechar</button></div>
      </div>

      <form id="prodForm">
        <div class="row">
          <div class="col">
            <label class="small">Nome do produto</label>
            <input id="p_name" required />
          </div>
          <div style="width:180px">
            <label class="small">SKU</label>
            <input id="p_sku" />
          </div>
        </div>

        <div class="row" style="margin-top:8px">
          <div class="col">
            <label class="small">Categoria</label>
            <input id="p_cat" placeholder="Ex: Hortifruti, Padaria" />
          </div>
          <div style="width:160px">
            <label class="small">Unidade</label>
            <input id="p_unit" placeholder="un, kg, cx" />
          </div>
          <div style="width:140px">
            <label class="small">Estoque inicial</label>
            <input id="p_qty" type="number" min="0" value="0" />
          </div>
        </div>

        <div class="row" style="margin-top:8px">
          <div class="col">
            <label class="small">EAN / Código de barras</label>
            <input id="p_ean" />
          </div>
          <div style="width:160px">
            <label class="small">Ponto de ressuprimento</label>
            <input id="p_min" type="number" min="0" value="1" />
          </div>
        </div>

        <div style="margin-top:10px;display:flex;gap:8px;align-items:center">
          <button class="btn" type="submit" id="saveProd">Salvar</button>
          <button class="btn ghost" id="saveAndAddLot" type="button">Salvar + Adicionar Lote</button>
          <div class="muted" style="margin-left:auto">Campos opcionais: EAN, Categoria</div>
        </div>
      </form>

      <hr style="margin:12px 0" />

      <div id="lotSection" style="display:none">
        <h4>Lotes e Validades</h4>
        <div style="display:flex;gap:8px;align-items:center;margin-bottom:8px">
          <input id="lot_code" placeholder="Código do lote" />
          <input id="lot_qty" type="number" min="1" placeholder="Qtd" style="width:100px" />
          <input id="lot_exp" type="date" style="width:160px" />
          <button class="btn" id="addLotBtn">Adicionar Lote</button>
        </div>
        <div style="max-height:160px;overflow:auto">
          <table id="lotTable"><thead><tr><th>Lote</th><th>Qtd</th><th>Validade</th></tr></thead><tbody></tbody></table>
        </div>
      </div>

    </div>
  </div>

  <footer>Feito para uso interno. Você pode baixar o HTML e abrir no navegador. Para recursos reais (impressão de etiquetas, integração PDV) é recomendado um backend.</footer>

  <script>
    // Simple stock manager - single-file app using localStorage.
    // Data structure: products: [{id, name, sku, ean, category, unit, qty, min, lots: [{code, qty, expiry}], movements: [{type,qty,date,reason}]}]

    const KEY = 'stock_manager_v1'
    const defaultDemo = [
      {id:genId(),name:'Maçã Gala',sku:'MAC-GALA',ean:'7890001112223',category:'Hortifruti',unit:'kg',qty:25,min:5,lots:[{code:'L001',qty:10,expiry:isoAddDays(6)},{code:'L002',qty:15,expiry:isoAddDays(40)}],movements:[]},
      {id:genId(),name:'Pão Francês (pacote)',sku:'PAO-FR',ean:'7890003334445',category:'Padaria',unit:'cx',qty:40,min:8,lots:[{code:'BREAD1',qty:40,expiry:isoAddDays(2)}],movements:[]},
      {id:genId(),name:'Arroz Branco 5kg',sku:'ARZ-5KG',ean:'7890005556667',category:'Mercearia',unit:'un',qty:18,min:4,lots:[],movements:[]}
    ]

    // Utils
    function genId(){return 'p_'+Math.random().toString(36).slice(2,9)}
    function isoAddDays(d){const t=new Date();t.setDate(t.getDate()+d);return t.toISOString().slice(0,10)}
    function load(){try{const raw=localStorage.getItem(KEY);return raw?JSON.parse(raw):null}catch(e){return null}}
    function save(data){localStorage.setItem(KEY,JSON.stringify(data))}

    // State
    let state = load() || {products:defaultDemo, movements:[]}
    if(!load()) save(state)

    // DOM refs
    const tbody = document.querySelector('#prodTable tbody')
    const kCount = document.getElementById('kCount')
    const kLow = document.getElementById('kLow')
    const alerts = document.getElementById('alerts')
    const search = document.getElementById('search')
    const filterCat = document.getElementById('filterCat')

    // Modal refs
    const modalBack = document.getElementById('modalBack')
    const openAdd = document.getElementById('openAdd')
    const closeModal = document.getElementById('closeModal')
    const prodForm = document.getElementById('prodForm')
    const modalTitle = document.getElementById('modalTitle')
    const lotSection = document.getElementById('lotSection')
    const lotTableBody = document.querySelector('#lotTable tbody')

    // Form fields
    const p_name = document.getElementById('p_name')
    const p_sku = document.getElementById('p_sku')
    const p_cat = document.getElementById('p_cat')
    const p_unit = document.getElementById('p_unit')
    const p_qty = document.getElementById('p_qty')
    const p_ean = document.getElementById('p_ean')
    const p_min = document.getElementById('p_min')
    const saveProd = document.getElementById('saveProd')
    const saveAndAddLot = document.getElementById('saveAndAddLot')

    // Lot fields
    const lot_code = document.getElementById('lot_code')
    const lot_qty = document.getElementById('lot_qty')
    const lot_exp = document.getElementById('lot_exp')
    const addLotBtn = document.getElementById('addLotBtn')

    let editingId = null

    // Quick receive/withdraw refs
    const rc_sku = document.getElementById('rc_sku')
    const rc_qty = document.getElementById('rc_qty')
    const rc_add = document.getElementById('rc_add')
    const out_sku = document.getElementById('out_sku')
    const out_qty = document.getElementById('out_qty')
    const out_sub = document.getElementById('out_sub')

    // CSV
    const exportCSV = document.getElementById('exportCSV')
    const importBtn = document.getElementById('importBtn')
    const importFile = document.getElementById('importFile')

    // Init
    render()
    populateFilter()

    // Event Listeners
    openAdd.addEventListener('click', ()=>{openModal()})
    closeModal.addEventListener('click', ()=>closeModalFn())
    modalBack.addEventListener('click', (e)=>{if(e.target===modalBack) closeModalFn()})

    prodForm.addEventListener('submit', (e)=>{e.preventDefault(); saveProduct();})
    saveAndAddLot.addEventListener('click', ()=>{saveProduct(true)})
    addLotBtn.addEventListener('click', addLotToEditing)

    rc_add.addEventListener('click', ()=>{quickReceive(rc_sku.value.trim(), Number(rc_qty.value) || 0)})
    out_sub.addEventListener('click', ()=>{quickOut(out_sku.value.trim(), Number(out_qty.value) || 0)})

    search.addEventListener('input', render)
    filterCat.addEventListener('change', render)

    exportCSV.addEventListener('click', ()=>{downloadCSV()})
    importBtn.addEventListener('click', ()=>importFile.click())
    importFile.addEventListener('change', handleImportFile)

    document.getElementById('openInventory').addEventListener('click', ()=>startInventory())
    document.getElementById('resetDemo').addEventListener('click', ()=>{if(confirm('Resetar dados para demo?')){state={products:defaultDemo,movements:[]};save(state);render();populateFilter();alert('Reset concluído')}})

    document.getElementById('reportLow').addEventListener('click', ()=>showReportLow())
    document.getElementById('reportExpiry').addEventListener('click', ()=>showReportExpiry())
    document.getElementById('reportMovement').addEventListener('click', ()=>showReportMovements())

    // Functions
    function render(){
      const q = search.value.trim().toLowerCase()
      const cat = filterCat.value
      tbody.innerHTML = ''
      state.products.forEach(p=>{
        if(cat && p.category!==cat) return
        if(q && !(`${p.name} ${p.sku||''} ${p.ean||''}`).toLowerCase().includes(q)) return
        const tr = document.createElement('tr')
        const soon = soonExpiry(p)
        tr.innerHTML = `
          <td><strong>${escapeHtml(p.name)}</strong><div class="small">${escapeHtml(p.category||'')}</div></td>
          <td class="small">${escapeHtml(p.sku||'')} ${p.ean?'<div class="small">EAN:'+escapeHtml(p.ean)+'</div>':''}</td>
          <td>${p.qty} <div class="small">min ${p.min||0} ${p.unit?('/'+p.unit):''}</div></td>
          <td class="small">${soon?'<span class="danger-txt">'+soon+'</span>':(p.lots && p.lots.length?listExpiry(p.lots):'—')}</td>
          <td style="text-align:right">
            <div class="flex" style="justify-content:flex-end">
              <button class="btn ghost" data-action="edit" data-id="${p.id}">Editar</button>
              <button class="btn" data-action="move" data-id="${p.id}">Movs</button>
              <button class="btn danger" data-action="del" data-id="${p.id}">Excluir</button>
            </div>
          </td>`
        tbody.appendChild(tr)
      })
      kCount.textContent = state.products.length
      const low = state.products.filter(p=>p.qty <= (p.min||0)).length
      kLow.textContent = low
      renderAlerts()
      addTableListeners()
    }

    function renderAlerts(){
      alerts.innerHTML = ''
      const lowItems = state.products.filter(p=>p.qty <= (p.min||0))
      if(lowItems.length){
        const h = document.createElement('div')
        h.innerHTML = '<strong>Estoque baixo</strong>'
        alerts.appendChild(h)
        lowItems.slice(0,6).forEach(p=>{
          const d = document.createElement('div')
          d.className='small'
          d.textContent = `${p.name} — ${p.qty} (min ${p.min||0})`
          alerts.appendChild(d)
        })
      }
      // expiry
      const soon = state.products.filter(p=>hasSoonExpiry(p,30))
      if(soon.length){
        const h2 = document.createElement('div')
        h2.style.marginTop='10px'
        h2.innerHTML = '<strong>Vencimentos próximos</strong>'
        alerts.appendChild(h2)
        soon.slice(0,6).forEach(p=>{
          const d = document.createElement('div')
          d.className='small'
          d.textContent = `${p.name} — ${soonExpiry(p)}`
          alerts.appendChild(d)
        })
      }
    }

    function addTableListeners(){
      tbody.querySelectorAll('button').forEach(btn=>{
        const action = btn.dataset.action
        const id = btn.dataset.id
        btn.onclick = ()=>{
          if(action==='edit') openModal(id)
          if(action==='del'){if(confirm('Excluir produto?')){deleteProduct(id)}}
          if(action==='move') showMovements(id)
        }
      })
    }

    function openModal(id=null){
      editingId = id
      modalBack.style.display='flex'
      lotSection.style.display='none'
      lotTableBody.innerHTML=''
      modalTitle.textContent = id? 'Editar Produto' : 'Novo Produto'
      if(id){
        const p = state.products.find(x=>x.id===id)
        if(!p) return
        p_name.value = p.name
        p_sku.value = p.sku||''
        p_cat.value = p.category||''
        p_unit.value = p.unit||''
        p_qty.value = p.qty||0
        p_ean.value = p.ean||''
        p_min.value = p.min||1
        if(p.lots && p.lots.length){
          lotSection.style.display='block'
          renderLotTable(p.lots)
        }
      } else {
        p_name.value=''
        p_sku.value=''
        p_cat.value=''
        p_unit.value=''
        p_qty.value=0
        p_ean.value=''
        p_min.value=1
      }
    }

    function closeModalFn(){ modalBack.style.display='none'; editingId=null }

    function saveProduct(openAddLot=false){
      const name = p_name.value.trim()
      if(!name) return alert('Nome é obrigatório')
      if(editingId){
        const p = state.products.find(x=>x.id===editingId)
        Object.assign(p,{name,sku:p_sku.value.trim(),category:p_cat.value.trim(),unit:p_unit.value.trim(),qty:Number(p_qty.value)||0,ean:p_ean.value.trim(),min:Number(p_min.value)||0})
      } else {
        const newP = {id:genId(),name,sku:p_sku.value.trim(),category:p_cat.value.trim(),unit:p_unit.value.trim(),qty:Number(p_qty.value)||0,ean:p_ean.value.trim(),min:Number(p_min.value)||0,lots:[],movements:[]}
        state.products.push(newP)
        editingId = newP.id
      }
      save(state)
      render()
      populateFilter()
      if(openAddLot){ lotSection.style.display='block' }
      else closeModalFn()
    }

    function deleteProduct(id){ state.products = state.products.filter(p=>p.id!==id); save(state); render(); populateFilter() }

    function addLotToEditing(){
      if(!editingId) return alert('Salve o produto primeiro')
      const p = state.products.find(x=>x.id===editingId)
      const code = lot_code.value.trim()||('L'+Math.random().toString(36).slice(2,6))
      const qty = Number(lot_qty.value)||0
      const expiry = lot_exp.value||''
      if(!qty) return alert('Quantidade do lote necessária')
      p.lots = p.lots || []
      p.lots.push({code,qty,expiry})
      p.qty = (p.qty||0) + qty
      lot_code.value=''
      lot_qty.value=''
      lot_exp.value=''
      renderLotTable(p.lots)
      save(state); render()
    }

    function renderLotTable(lots){
      lotTableBody.innerHTML=''
      lots.forEach(l=>{
        const tr = document.createElement('tr')
        tr.innerHTML = `<td>${escapeHtml(l.code)}</td><td>${l.qty}</td><td>${l.expiry||'—'}</td>`
        lotTableBody.appendChild(tr)
      })
    }

    function quickReceive(code, qty){
      if(!code || !qty) return alert('SKU/EAN e quantidade necessários')
      const p = findByCode(code)
      if(!p) return alert('Produto não encontrado')
      p.qty += qty
      p.movements = p.movements||[]
      p.movements.push({type:'entrada',qty,date:new Date().toISOString(),reason:'Recebimento Rápido'})
      save(state); render(); alert('Recebimento registrado')
    }

    function quickOut(code, qty){
      if(!code || !qty) return alert('SKU/EAN e quantidade necessários')
      const p = findByCode(code)
      if(!p) return alert('Produto não encontrado')
      if(p.qty < qty) return alert('Quantidade insuficiente')
      p.qty -= qty
      p.movements = p.movements||[]
      p.movements.push({type:'saida',qty,date:new Date().toISOString(),reason:'Saída Rápida'})
      save(state); render(); alert('Saída registrada')
    }

    function findByCode(code){
      const c = code.toLowerCase()
      return state.products.find(p=> (p.sku||'').toLowerCase()===c || (p.ean||'').toLowerCase()===c || (p.id||'').toLowerCase()===c || p.name.toLowerCase()===c )
    }

    function populateFilter(){
      const cats = Array.from(new Set(state.products.map(p=>p.category).filter(Boolean)))
      filterCat.innerHTML = '<option value="">Todas categorias</option>' + cats.map(c=>`<option value="${escapeHtml(c)}">${escapeHtml(c)}</option>`).join('')
    }

    function soonExpiry(p){
      if(!p.lots || !p.lots.length) return ''
      const soon = p.lots.map(l=>l.expiry).filter(Boolean).sort()[0]
      if(!soon) return ''
      const diff = daysBetween(new Date(), new Date(soon))
      if(diff<0) return 'vencido'
      if(diff<=7) return `vence em ${diff} dia(s)`
      if(diff<=30) return `vence em ${diff} dias`
      return ''
    }

    function listExpiry(lots){
      const items = lots.filter(l=>l.expiry).slice(0,3).map(l=>`${l.code}:${l.expiry}`)
      return items.join(', ')
    }

    function daysBetween(a,b){return Math.ceil((b-a)/(1000*60*60*24))}
    function hasSoonExpiry(p,days){return (p.lots||[]).some(l=>l.expiry && daysBetween(new Date(),new Date(l.expiry))<=days)}

    // Movement viewer
    function showMovements(id){
      const p = state.products.find(x=>x.id===id)
      if(!p) return
      const lines = (p.movements||[]).slice().reverse().map(m=>`${m.date.split('T')[0]} — ${m.type} — ${m.qty} — ${m.reason||''}`).join('\n') || 'Sem movimentos'
      alert(`Movimentos de ${p.name}:\n\n`+lines)
    }

    function showReportLow(){
      const low = state.products.filter(p=>p.qty <= (p.min||0))
      if(!low.length) return alert('Nenhum produto com estoque baixo')
      const text = low.map(p=>`${p.name} — ${p.qty} (min ${p.min||0})`).join('\n')
      alert('Produtos com estoque baixo:\n\n'+text)
    }

    function showReportExpiry(){
      const soon = state.products.filter(p=>hasSoonExpiry(p,30))
      if(!soon.length) return alert('Nenhum vencimento próximo (30 dias)')
      const text = soon.map(p=>`${p.name} — ${soonExpiry(p)}`).join('\n')
      alert('Próximos do vencimento (30d):\n\n'+text)
    }

    function showReportMovements(){
      const last30 = new Date(); last30.setDate(last30.getDate()-30)
      const lines = []
      state.products.forEach(p=>{
        (p.movements||[]).forEach(m=>{if(new Date(m.date) >= last30) lines.push(`${m.date.split('T')[0]} — ${p.name} — ${m.type} — ${m.qty} — ${m.reason||''}`)})
      })
      if(!lines.length) return alert('Sem movimentações nos últimos 30 dias')
      alert('Movimentações (30d):\n\n'+lines.sort().join('\n'))
    }

    // CSV export/import
    function downloadCSV(){
      const rows = []
      rows.push(['id','name','sku','ean','category','unit','qty','min','lots'])
      state.products.forEach(p=>{
        const lots = (p.lots||[]).map(l=>`${l.code}|${l.qty}|${l.expiry||''}`).join(';;')
        rows.push([p.id,p.name,p.sku||'',p.ean||'',p.category||'',p.unit||'',p.qty||0,p.min||0,lots])
      })
      const csv = rows.map(r=>r.map(c=>`"${String(c).replace(/"/g,'""')}"`).join(',')).join('\n')
      const blob = new Blob([csv],{type:'text/csv;charset=utf-8;'})
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a');a.href=url;a.download='estoque_export.csv';a.click();URL.revokeObjectURL(url)
    }

    function handleImportFile(e){
      const f = e.target.files[0]
      if(!f) return
      const reader = new FileReader()
      reader.onload = ()=>{
        try{const txt = reader.result; parseCSVImport(txt); importFile.value=''}catch(err){alert('Erro ao importar')}
      }
      reader.readAsText(f,'UTF-8')
    }

    function parseCSVImport(txt){
      const lines = txt.split(/\r?\n/).filter(Boolean)
      const headers = lines.shift().split(',').map(h=>h.replace(/(^\")|("$)/g,''))
      const idx = (k)=>headers.indexOf(k)
      if(idx('name')<0) return alert('CSV inválido — precisa ter coluna name')
      const imported = lines.map(l=>{const cols = splitCSV(l);return {id:cols[idx('id')]||genId(),name:cols[idx('name')],sku:cols[idx('sku')]||'',ean:cols[idx('ean')]||'',category:cols[idx('category')]||'',unit:cols[idx('unit')]||'',qty:Number(cols[idx('qty')]||0),min:Number(cols[idx('min')]||0),lots:parseLots(cols[idx('lots')]||'')}})
      // merge: update existing by id or push new
      imported.forEach(ip=>{
        const existing = state.products.find(p=>p.id===ip.id || (p.sku && ip.sku && p.sku===ip.sku))
        if(existing){Object.assign(existing,ip)} else state.products.push(ip)
      })
      save(state); render(); populateFilter(); alert('Importação concluída')
    }

    function parseLots(txt){
      if(!txt) return []
      return txt.split(';;').map(s=>{const parts=s.split('|');return {code:parts[0]||'',qty:Number(parts[1]||0),expiry:parts[2]||''}})
    }

    // naive CSV splitter (handles quoted)
    function splitCSV(line){
      const res=[]; let cur=''; let inQ=false
      for(let i=0;i<line.length;i++){const ch=line[i]
        if(ch==='"'){inQ=!inQ;continue}
        if(ch===',' && !inQ){res.push(cur);cur='';continue}
        cur+=ch
      }
      res.push(cur)
      return res.map(s=>s.replace(/^"|"$/g,''))
    }

    // helpers
    function escapeHtml(s){if(!s && s!==0) return ''; return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;')}

    // Simple Inventory flow: counts by user input (bulk) — quick partial implementation
    function startInventory(){
      const list = state.products.map(p=>`${p.id};${p.name};${p.qty}`).join('\n')
      const txt = prompt('Inventário rápido:\nCole/edite linhas no formato: id;nome;contagem\n(Exemplo já existente)\n\n'+list)
      if(!txt) return
      const lines = txt.split(/\r?\n/).filter(Boolean)
      lines.forEach(l=>{const parts=l.split(';'); if(parts.length>=3){const id=parts[0].trim(); const c=Number(parts[2])||0; const p=state.products.find(x=>x.id===id); if(p){p.qty=c; p.movements = p.movements||[]; p.movements.push({type:'ajuste',qty:c,date:new Date().toISOString(),reason:'Inventário rápido'})}}})
      save(state); render(); alert('Inventário salvo')
    }

    // small helpers for UI actions
    function escapeAttr(s){return String(s).replace(/"/g,'&quot;')}

  </script>
</body>
</html>
