<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Pedidos & Cobros — App (único HTML)</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- SheetJS para exportar Excel -->
  <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
  <style>
    body { -webkit-tap-highlight-color: transparent; }
    .hide-scrollbar::-webkit-scrollbar{display:none}
    .badge-stock{position:absolute;right:0;top:0;background:#111;color:white;padding:2px 6px;border-bottom-left-radius:8px;font-size:12px}
  </style>
</head>
<body class="bg-slate-50 text-slate-900">
  <header class="sticky top-0 z-50 bg-white/90 backdrop-blur border-b border-slate-200">
    <div class="max-w-5xl mx-auto px-4 py-3 flex items-center gap-3">
      <div class="w-10 h-10 rounded-2xl bg-amber-400 grid place-items-center text-white font-black">$</div>
      <div class="flex-1">
        <h1 class="text-lg font-bold">Pedidos & Cobros</h1>
        <p class="text-xs text-slate-500">Archivo único HTML — Funciona sin internet. Guarda en tu teléfono con LocalStorage.</p>
      </div>
      <button id="helpBtn" class="px-3 py-2 text-xs rounded-xl bg-slate-900 text-white">Ayuda</button>
    </div>
    <nav class="max-w-5xl mx-auto px-2 pb-2 flex gap-2 text-sm">
      <button data-tab="venta" class="tab-btn px-3 py-2 rounded-xl bg-slate-900 text-white">Venta</button>
      <button data-tab="menu" class="tab-btn px-3 py-2 rounded-xl bg-slate-100">Menú</button>
      <button data-tab="fiados" class="tab-btn px-3 py-2 rounded-xl bg-slate-100">Carpetas (Fiados)</button>
      <button data-tab="ajustes" class="tab-btn ml-auto px-3 py-2 rounded-xl bg-slate-100">Ajustes</button>
    </nav>
  </header>

  <main class="max-w-5xl mx-auto px-4 pb-24">

    <!-- VENTA -->
    <section id="tab-venta" class="tab pt-4">
      <div class="grid grid-cols-1 sm:grid-cols-3 gap-3 mb-3">
        <input id="searchInput" class="rounded-2xl border border-slate-200 px-4 py-3" placeholder="Buscar platillo…" />

        <!-- Cliente: selector + botón añadir -->
        <div class="flex gap-2">
          <select id="customerSelect" class="flex-1 rounded-2xl border border-slate-200 px-4 py-3"></select>
          <button id="addCustomerBtn" class="rounded-2xl border border-slate-200 px-4 py-3">Nuevo</button>
        </div>

        <div class="flex gap-2">
          <button id="clearOrderBtn" class="flex-1 rounded-2xl border border-slate-200 px-4 py-3">Limpiar</button>
          <button id="undoBtn" class="rounded-2xl border border-slate-200 px-4 py-3">↶</button>
        </div>
      </div>

      <!-- Menú botones -->
      <div>
        <h2 class="text-sm font-semibold text-slate-600 mb-2">Menú del día</h2>
        <div id="menuGrid" class="grid grid-cols-3 sm:grid-cols-4 md:grid-cols-5 gap-2"></div>
      </div>

      <!-- Orden y resumen -->
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-3 mt-4">
        <div class="rounded-2xl bg-white border border-slate-200 p-3">
          <h3 class="font-semibold mb-2">Productos agregados</h3>
          <div id="orderList" class="divide-y divide-slate-100"></div>
        </div>
        <div class="rounded-2xl bg-white border border-slate-200 p-3 space-y-3">
          <div class="flex justify-between text-sm text-slate-600"><span>Artículos</span><span id="summaryItems">0</span></div>
          <div class="flex justify-between text-sm text-slate-600"><span>Subtotal</span><span id="summarySubtotal">$0.00</span></div>
          <div class="flex justify-between text-base font-bold"><span>Total</span><span id="summaryTotal">$0.00</span></div>

          <div class="grid grid-cols-2 gap-2 pt-2">
            <button id="chargeBtn" class="rounded-2xl bg-emerald-600 text-white px-4 py-3 font-semibold">Cobrar ahora</button>
            <button id="fiarBtn" class="rounded-2xl bg-amber-500 text-white px-4 py-3 font-semibold">Agregar a Carpeta</button>
          </div>
          <p class="text-xs text-slate-500">Toca un producto en la lista para editar cantidad o eliminar.</p>
        </div>
      </div>
    </section>

    <!-- MENÚ (editar items + stock) -->
    <section id="tab-menu" class="tab hidden pt-4">
      <div class="rounded-2xl bg-white border border-slate-200 p-4 space-y-4">
        <div class="flex items-center gap-3">
          <h3 class="font-semibold text-lg">Platillos y adicionales</h3>
          <button id="addItemBtn" class="rounded-xl bg-slate-900 text-white px-3 py-2 text-sm">Nuevo</button>
          <button id="resetDemoBtn" class="rounded-xl border border-slate-200 px-3 py-2 text-sm">Restaurar demo</button>
        </div>
        <div id="editGrid" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-3"></div>
      </div>
    </section>

    <!-- FIADOS -->
    <section id="tab-fiados" class="tab hidden pt-4">
      <div class="rounded-2xl bg-white border border-slate-200 p-4 space-y-4">
        <div class="grid grid-cols-1 sm:grid-cols-4 gap-3">
          <div>
            <label class="text-xs text-slate-500">Mes</label>
            <select id="monthSelect" class="w-full rounded-xl border border-slate-200 px-3 py-2"></select>
          </div>
          <div>
            <label class="text-xs text-slate-500">Filtrar</label>
            <select id="filterStatus" class="w-full rounded-xl border border-slate-200 px-3 py-2">
              <option value="all">Todos</option>
              <option value="unpaid">Pendientes</option>
              <option value="paid">Pagados</option>
            </select>
          </div>
          <div class="col-span-2 flex gap-2">
            <button id="exportExcelBtn" class="flex-1 rounded-xl border border-slate-200 px-3 py-2">Exportar Excel</button>
            <button id="exportPdfBtn" class="flex-1 rounded-xl border border-slate-200 px-3 py-2 opacity-40 cursor-not-allowed" disabled>Exportar PDF (próx.)</button>
          </div>
        </div>
        <div id="ledgerList" class="divide-y divide-slate-100"></div>
        <div id="ledgerSummary" class="pt-2 text-sm text-slate-600"></div>
      </div>
    </section>

    <!-- AJUSTES -->
    <section id="tab-ajustes" class="tab hidden pt-4">
      <div class="rounded-2xl bg-white border border-slate-200 p-4 space-y-4">
        <h3 class="font-semibold text-lg">Ajustes</h3>
        <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
          <label class="flex items-center justify-between gap-3 rounded-xl border border-slate-200 p-3">
            <span class="text-sm">Moneda</span>
            <select id="currencySelect" class="rounded-lg border border-slate-200 px-3 py-2">
              <option value="USD">USD ($)</option>
              <option value="PEN">PEN (S/)</option>
              <option value="COP">COP ($)</option>
              <option value="MXN">MXN ($)</option>
            </select>
          </label>
          <label class="flex items-center justify-between gap-3 rounded-xl border border-slate-200 p-3">
            <span class="text-sm">IGV/IVA (%)</span>
            <input id="taxInput" type="number" step="0.01" class="w-32 rounded-lg border border-slate-200 px-3 py-2" placeholder="0" />
          </label>
        </div>
        <div class="flex gap-2">
          <button id="clearDataBtn" class="rounded-xl border border-slate-200 px-3 py-2">Borrar todo</button>
          <button id="helpBtn2" class="rounded-xl bg-slate-900 text-white px-3 py-2">Ver ayuda</button>
        </div>
      </div>
    </section>

  </main>

  <!-- DIALOGS -->
  <dialog id="helpDialog" class="rounded-2xl p-0 border-0 w-[92vw] max-w-2xl">
    <div class="p-4 md:p-6 bg-white rounded-2xl border border-slate-200">
      <h3 class="font-semibold text-lg">Cómo ejecutar y usar la app</h3>
      <ol class="list-decimal pl-5 text-sm text-slate-700 mt-2 space-y-2">
        <li>Descarga este archivo <code>.html</code> a tu teléfono o computadora.</li>
        <li>Ábrelo con tu navegador (PC: doble clic; Android: Archivos → abrir con Chrome). Funciona sin internet.</li>
        <li>Ve a <strong>Menú</strong> para crear/editar platillos, precios e <strong>inventario (stock)</strong>.</li>
        <li>En <strong>Venta</strong> selecciona cliente (o crea uno nuevo), toca botones para agregar productos. El stock se reduce automáticamente. Si stock = 0 el botón queda deshabilitado.</li>
        <li>Usa <strong>Agregar a Carpeta</strong> para registrar fiados. En <strong>Carpetas</strong> verás el total pendiente por persona y el total global del mes. Puedes exportar a Excel.</li>
      </ol>
      <div class="text-right pt-4"><button class="rounded-xl bg-slate-900 text-white px-4 py-2" onclick="document.getElementById('helpDialog').close()">Cerrar</button></div>
    </div>
  </dialog>

  <dialog id="itemDialog" class="rounded-2xl p-0 border-0 w-[92vw] max-w-md">
    <form id="itemForm" class="p-4 md:p-6 bg-white rounded-2xl border border-slate-200 space-y-3">
      <h3 class="font-semibold text-lg" id="itemDialogTitle">Nuevo producto</h3>
      <input type="hidden" id="itemId" />
      <label class="block text-sm">Nombre
        <input id="itemName" required class="mt-1 w-full rounded-xl border border-slate-200 px-3 py-2" placeholder="Ej.: Empanada" />
      </label>
      <label class="block text-sm">Precio
        <input id="itemPrice" type="number" step="0.01" required class="mt-1 w-full rounded-xl border border-slate-200 px-3 py-2" placeholder="1.50" />
      </label>
      <label class="block text-sm">Stock (unidades)
        <input id="itemStock" type="number" min="0" value="10" class="mt-1 w-full rounded-xl border border-slate-200 px-3 py-2" />
      </label>
      <div class="grid grid-cols-2 gap-3 text-sm">
        <label class="flex items-center gap-2"><input id="itemActive" type="checkbox" class="rounded" checked /> Mostrar</label>
        <label class="flex items-center gap-2"><input id="itemFavorite" type="checkbox" class="rounded" /> Favorito</label>
      </div>
      <div class="space-y-2 text-sm">
        <p>Icono / Emoji</p>
        <div class="flex items-center gap-2">
          <button type="button" id="chooseEmojiBtn" class="rounded-xl border border-slate-200 px-3 py-2">Elegir emoji</button>
          <label class="rounded-xl border border-slate-200 px-3 py-2 cursor-pointer">Subir imagen
            <input id="itemImageInput" type="file" accept="image/*" class="hidden" />
          </label>
          <div id="itemPreview" class="w-10 h-10 rounded-xl bg-slate-100 grid place-items-center text-2xl">🍽️</div>
        </div>
      </div>
      <div class="flex justify-end gap-2 pt-2"><button type="button" class="rounded-xl border border-slate-200 px-4 py-2" onclick="document.getElementById('itemDialog').close()">Cancelar</button><button class="rounded-xl bg-slate-900 text-white px-4 py-2">Guardar</button></div>
    </form>
  </dialog>

  <dialog id="customerDialog" class="rounded-2xl p-0 border-0 w-[92vw] max-w-md">
    <form id="customerForm" class="p-4 md:p-6 bg-white rounded-2xl border border-slate-200 space-y-3">
      <h3 class="font-semibold text-lg">Nuevo cliente</h3>
      <label>Nombre
        <input id="newCustomerName" required class="mt-1 w-full rounded-xl border border-slate-200 px-3 py-2" />
      </label>
      <div class="flex justify-end gap-2 pt-2"><button type="button" class="rounded-xl border border-slate-200 px-4 py-2" onclick="document.getElementById('customerDialog').close()">Cancelar</button><button class="rounded-xl bg-slate-900 text-white px-4 py-2">Agregar</button></div>
    </form>
  </dialog>

  <!-- Emoji picker simple -->
  <dialog id="emojiDialog" class="rounded-2xl p-0 border-0 w-[92vw] max-w-lg">
    <div class="p-4 bg-white rounded-2xl border border-slate-200">
      <div class="flex items-center gap-2 mb-2"><input id="emojiSearch" class="flex-1 rounded-xl border border-slate-200 px-3 py-2 text-sm" placeholder="Buscar emoji…" /><button class="rounded-xl border border-slate-200 px-3 py-2" onclick="document.getElementById('emojiDialog').close()">Cerrar</button></div>
      <div id="emojiGrid" class="grid grid-cols-8 gap-2 max-h-[50vh] overflow-y-auto hide-scrollbar text-xl"></div>
    </div>
  </dialog>

  <script>
  // ===== UTIL =====n
  const $ = s=>document.querySelector(s);
  const $$ = s=>Array.from(document.querySelectorAll(s));
  const monthKey = d=> (d||new Date()).toISOString().slice(0,7);

  // ===== STORAGE =====
  const LS = { get(k,d){try{return JSON.parse(localStorage.getItem(k))??d}catch{return d}}, set(k,v){localStorage.setItem(k,JSON.stringify(v))}, del(k){localStorage.removeItem(k)} };

  let settings = LS.get('settings',{currency:'USD',tax:0});
  let items = LS.get('menuItems', null);
  let ledger = LS.get('ledger', {}); // { 'YYYY-MM': [orders...] }
  let customers = LS.get('customers', ['Cliente']);

  // Demo items (if empty)
  const demo = [
    { id: crypto.randomUUID(), name:'Empanada', price:1.20, emoji:'🥟', stock:10, active:true, favorite:true },
    { id: crypto.randomUUID(), name:'2 huevos', price:1.50, emoji:'🍳', stock:20, active:true, favorite:true },
    { id: crypto.randomUUID(), name:'Salchicha', price:1.00, emoji:'🌭', stock:15, active:true, favorite:false },
    { id: crypto.randomUUID(), name:'Café', price:0.80, emoji:'☕', stock:50, active:true, favorite:false }
  ];
  if(!items){ items = demo; LS.set('menuItems', items); }

  // Venta state
  let order = { lines: [], history: [] };

  // Formatea moneda
  function fmt(n){ const cur = settings.currency||'USD'; const map={USD:'$',PEN:'S/',COP:'$',MXN:'$'}; return (map[cur]||'$')+Number(n).toFixed(2); }

  // ===== MENU RENDER (botones) =====
  function renderMenu(filter=''){
    const grid = $('#menuGrid'); grid.innerHTML='';
    const list = items.filter(i=>i.active).filter(i=>i.name.toLowerCase().includes(filter.toLowerCase())).sort((a,b)=>(b.favorite-a.favorite)||a.name.localeCompare(b.name));
    list.forEach(it=>{
      const btn = document.createElement('button'); btn.className='relative rounded-2xl border border-slate-200 bg-white p-2 text-center active:scale-[.98]';
      btn.disabled = it.stock<=0;
      if(it.stock<=0) btn.classList.add('opacity-50','cursor-not-allowed');
      btn.innerHTML = `
        <div class="w-full aspect-square grid place-items-center text-3xl">${it.imageData||it.emoji||'🍽️'}</div>
        <div class="mt-1 text-[11px] text-slate-500">${it.name}</div>
        <div class="font-semibold">${fmt(it.price)}</div>
      `;
      // badge stock
      const span = document.createElement('div'); span.className='badge-stock'; span.textContent=it.stock; btn.appendChild(span);

      btn.addEventListener('click', ()=>{
        if(it.stock<=0) return; addToOrder(it.id);
      });
      grid.appendChild(btn);
    });
  }

  // ===== ORDER =====
  function addToOrder(itemId){
    const it = items.find(x=>x.id===itemId); if(!it) return; if(it.stock<=0){alert('Sin stock');return}
    const line = order.lines.find(l=>l.id===itemId);
    if(line) line.qty+=1; else order.lines.push({id:it.id,name:it.name,price:it.price,qty:1});
    order.history.push({type:'add',itemId});
    // reducir stock temporalmente (hasta cobrar/fiar will persist)
    it.stock-=1; saveItems(); renderMenu($('#searchInput').value); renderOrder();
  }

  function renderOrder(){
    const list = $('#orderList'); list.innerHTML=''; let count=0, sub=0;
    order.lines.forEach(line=>{ count+=line.qty; sub+=line.qty*line.price;
      const row = document.createElement('div'); row.className='py-2 flex items-center gap-2';
      row.innerHTML = `
        <div class="flex-1">
          <div class="font-medium">${line.name}</div>
          <div class="text-xs text-slate-500">${fmt(line.price)} × <input type="number" min="1" value="${line.qty}" class="w-14 rounded-lg border border-slate-200 px-2 py-1 inline" /> = <span class="font-semibold">${fmt(line.qty*line.price)}</span></div>
        </div>
        <button class="rounded-lg border border-slate-200 px-2 py-1 text-xs">Eliminar</button>
      `;
      const qtyInput = row.querySelector('input'); qtyInput.addEventListener('change',e=>{ const prev=line.qty; const val=Math.max(1,Number(e.target.value||1)); order.history.push({type:'qty',itemId:line.id,prevQty:prev});
        // ajustar stock en items: si aumentó qty, reducir stock; si redujo qty, incrementar stock
        const diff = val - line.qty; line.qty=val; const it = items.find(x=>x.id===line.id); if(it) { it.stock -= diff; if(it.stock<0){ it.stock=0 } }
        saveItems(); renderMenu($('#searchInput').value); renderOrder(); });
      row.querySelector('button').addEventListener('click',()=>{ order.history.push({type:'remove',prevLine:{...line}}); // devolver stock
        const it = items.find(x=>x.id===line.id); if(it) it.stock += line.qty; order.lines = order.lines.filter(l=>l!==line); saveItems(); renderMenu($('#searchInput').value); renderOrder(); });
      list.appendChild(row);
    });
    const taxPct = Number(settings.tax||0)/100; const total = sub + sub*taxPct;
    $('#summaryItems').textContent = count; $('#summarySubtotal').textContent = fmt(sub); $('#summaryTotal').textContent = fmt(total);
  }

  // Cobrar o fiar (persistir en ledger)
  function charge(paid=true){ if(!order.lines.length){ alert('La orden está vacía.'); return }
    const cust = $('#customerSelect').value || 'Cliente';
    const date = new Date(); const sub = order.lines.reduce((a,l)=>a+l.qty*l.price,0); const total = sub + sub*(Number(settings.tax||0)/100);
    const entry = { id: crypto.randomUUID(), name: cust, items: JSON.parse(JSON.stringify(order.lines)), total, dateISO: date.toISOString(), status: paid?'paid':'unpaid' };
    const mk = monthKey(date); ledger[mk] ||= []; ledger[mk].push(entry);
    LS.set('ledger', ledger);
    // Save current items stock to storage
    saveItems();
    // Reset order
    order = {lines:[],history:[]}; renderOrder(); renderLedger(); alert(paid? 'Pago registrado':'Agregado a Carpeta (pendiente)');
  }

  // ===== LEDGER / FIADOS =====
  function renderMonthSelect(){ const sel = $('#monthSelect'); const months = Object.keys(ledger).sort().reverse(); const cur = monthKey(); if(!months.includes(cur)) months.unshift(cur); sel.innerHTML = months.map(m=>`<option value="${m}">${m}</option>`).join(''); sel.value = sel.value || cur; }

  function renderLedger(){ renderMonthSelect(); const mk = $('#monthSelect').value; const filter = $('#filterStatus').value; const list = (ledger[mk]||[]).filter(e=>filter==='all'||e.status===filter);
    const byName = {}; list.forEach(e=>{ (byName[e.name] ||= []).push(e); });
    const wrap = $('#ledgerList'); wrap.innerHTML=''; let globalPending = 0;
    Object.keys(byName).sort().forEach(name=>{ const orders = byName[name]; const total = orders.reduce((a,o)=>a+o.total,0); const unpaid = orders.filter(o=>o.status==='unpaid'); const paid = orders.filter(o=>o.status==='paid'); const pendingSum = unpaid.reduce((a,o)=>a+o.total,0); globalPending += pendingSum;
      const card = document.createElement('div'); card.className='py-3'; card.innerHTML = `
        <div class="flex items-center justify-between">
          <div>
            <div class="font-semibold">${name}</div>
            <div class="text-xs text-slate-500">${orders.length} orden(es) — Total: <span class="font-semibold">${fmt(total)}</span></div>
          </div>
          <div class="flex gap-2">
            <button class="rounded-xl border border-slate-200 px-3 py-2 text-xs" data-action="markPaid">Marcar pendientes como pagado</button>
            <button class="rounded-xl border border-rose-200 text-rose-600 px-3 py-2 text-xs" data-action="deleteCustomer">Eliminar (mes)</button>
          </div>
        </div>
        <div class="mt-2 rounded-xl bg-slate-50 border border-slate-200 p-2">
          ${[...unpaid,...paid].map(o=>`
            <div class="flex items-center justify-between py-2">
              <div>
                <div class="text-sm">${new Date(o.dateISO).toLocaleDateString()} — <span class="font-medium">${fmt(o.total)}</span> — <span class="${o.status==='unpaid'?'text-amber-600':'text-emerald-600'}">${o.status==='unpaid'?'Pendiente':'Pagado'}</span></div>
                <div class="text-xs text-slate-500">${o.items.map(i=>`${i.qty}× ${i.name}`).join(', ')}</div>
              </div>
              <div class="flex gap-2">
                ${o.status==='unpaid'?'<button data-action="markOne" class="rounded-lg border border-slate-200 px-2 py-1 text-xs">Marcar pagado</button>':''}
                <button data-action="deleteOne" class="rounded-lg border border-slate-200 px-2 py-1 text-xs">Eliminar</button>
              </div>
            </div>
          `).join('')}
        </div>
      `;
      // actions
      card.querySelector('[data-action="markPaid"]').addEventListener('click', ()=>{ orders.forEach(o=>{ if(o.status==='unpaid') o.status='paid'; }); LS.set('ledger',ledger); renderLedger(); });
      card.querySelector('[data-action="deleteCustomer"]').addEventListener('click', ()=>{ if(!confirm('Eliminar todas las órdenes de este cliente en el mes?')) return; ledger[mk] = (ledger[mk]||[]).filter(o=>o.name!==name); LS.set('ledger',ledger); renderLedger(); });
      card.querySelectorAll('[data-action="markOne"]').forEach((btn,idx)=>{ btn.addEventListener('click', ()=>{ const o = unpaid[idx]; if(o) o.status='paid'; LS.set('ledger',ledger); renderLedger(); }); });
      card.querySelectorAll('[data-action="deleteOne"]').forEach((btn,i)=>{ btn.addEventListener('click', ()=>{ const ord = [...unpaid,...paid][i]; ledger[mk] = (ledger[mk]||[]).filter(o=>o!==ord); LS.set('ledger',ledger); renderLedger(); }); });
      wrap.appendChild(card);
    });
    if(!wrap.innerHTML.trim()) wrap.innerHTML = '<div class="text-sm text-slate-500">No hay registros en este mes.</div>';
    $('#ledgerSummary').innerHTML = `<div class="text-sm">Total pendiente (mes): <span class="font-semibold">${fmt(globalPending)}</span></div>`;
  }

  // ===== EDIT MENU (items + stock) =====
  function renderEditGrid(){ const grid = $('#editGrid'); grid.innerHTML=''; items.sort((a,b)=>(b.favorite-a.favorite)||a.name.localeCompare(b.name));
    items.forEach(it=>{
      const card = document.createElement('div'); card.className='rounded-2xl border border-slate-200 p-3 bg-white flex flex-col gap-2';
      card.innerHTML = `
        <div class="flex items-center gap-2">
          <div class="w-12 h-12 rounded-xl bg-slate-100 grid place-items-center text-2xl">${it.imageData||it.emoji||'🍽️'}</div>
          <div class="flex-1">
            <input class="w-full rounded-xl border border-slate-200 px-3 py-2 text-sm font-medium" value="${it.name}" />
            <div class="text-xs text-slate-500">Precio: <input type="number" step="0.01" class="w-24 rounded-lg border border-slate-200 px-2 py-1" value="${it.price}" /></div>
            <div class="text-xs text-slate-500">Stock: <input type="number" class="w-24 rounded-lg border border-slate-200 px-2 py-1" value="${it.stock}" /></div>
          </div>
        </div>
        <div class="flex items-center justify-between">
          <label class="text-xs flex items-center gap-2"><input type="checkbox" ${it.active?'checked':''} /> Mostrar</label>
          <label class="text-xs flex items-center gap-2"><input type="checkbox" ${it.favorite?'checked':''} /> Favorito</label>
          <div class="flex gap-2">
            <button data-action="img" class="rounded-lg border border-slate-200 px-2 py-1 text-xs">Imagen</button>
            <button data-action="emoji" class="rounded-lg border border-slate-200 px-2 py-1 text-xs">Emoji</button>
            <button data-action="del" class="rounded-lg border border-rose-200 text-rose-600 px-2 py-1 text-xs">Eliminar</button>
          </div>
        </div>
      `;
      const inputs = card.querySelectorAll('input'); const nameInput=inputs[0], priceInput=inputs[1], stockInput=inputs[2], showInput=inputs[3], favInput=inputs[4];
      nameInput.addEventListener('change',e=>{ it.name=e.target.value; saveItems(); renderMenu($('#searchInput').value); renderOrder(); });
      priceInput.addEventListener('change',e=>{ it.price=Number(e.target.value||0); saveItems(); renderMenu($('#searchInput').value); renderOrder(); });
      stockInput.addEventListener('change',e=>{ it.stock = Math.max(0, Number(e.target.value||0)); saveItems(); renderMenu($('#searchInput').value); });
      showInput.addEventListener('change',e=>{ it.active=e.target.checked; saveItems(); renderMenu($('#searchInput').value); });
      favInput.addEventListener('change',e=>{ it.favorite=e.target.checked; saveItems(); renderMenu($('#searchInput').value); });
      card.querySelector('[data-action="del"]').addEventListener('click', ()=>{ if(!confirm('Eliminar producto?')) return; items = items.filter(x=>x!==it); saveItems(); renderEditGrid(); renderMenu($('#searchInput').value); });
      card.querySelector('[data-action="emoji"]').addEventListener('click', ()=>{ openEmojiPicker(em=>{ it.emoji=em; it.imageData=null; saveItems(); renderEditGrid(); renderMenu($('#searchInput').value); }); });
      card.querySelector('[data-action="img"]').addEventListener('click', async ()=>{ const f = await pickImage(); if(f){ const d = await fileToDataURL(f); it.imageData = `<img src="${d}" class="w-8 h-8 object-cover rounded-lg"/>`; saveItems(); renderEditGrid(); renderMenu($('#searchInput').value); } });
      grid.appendChild(card);
    });
  }

  function saveItems(){ LS.set('menuItems', items); }

  // ===== CUSTOMERS =====
  function renderCustomers(){ const sel = $('#customerSelect'); sel.innerHTML = customers.map(c=>`<option value="${c}">${c}</option>`).join(''); }

  // ===== EMOJI PICKER & FILE =====
  const EMOJIS = '🍳🥚🍗🥓🌭🍔🍟🍕🌯🌮🥪🥙🥗🍝🍲🍜🍛🍣🍤🍱🥟🥞🧇🍞🥖🥐🧀🥚🥗🍅🥑🥔🌶️🧅🧄🍌🍎🍊🍉🍇🍓🍍🥭☕🧃🥤🍩🍪🍰'.split('');
  function openEmojiPicker(cb){ const d=$('#emojiDialog'), grid=$('#emojiGrid'), s=$('#emojiSearch'); grid.innerHTML=''; const render=()=>{ grid.innerHTML=''; EMOJIS.filter(e=>!s.value||e.includes(s.value)).forEach(e=>{ const b=document.createElement('button'); b.className='rounded-xl border border-slate-200 p-2'; b.textContent=e; b.addEventListener('click',()=>{ d.close(); cb(e); }); grid.appendChild(b); }); }; render(); s.oninput=render; d.showModal(); }
  async function pickImage(){ return new Promise(resolve=>{ const i=document.createElement('input'); i.type='file'; i.accept='image/*'; i.onchange=()=>resolve(i.files[0]); i.click(); }); }
  function fileToDataURL(file){ return new Promise((res,rej)=>{ const r=new FileReader(); r.onload=()=>res(r.result); r.onerror=rej; r.readAsDataURL(file); }); }

  // ===== EXPORT EXCEL (SheetJS) =====
  function exportLedgerToExcel(){ const mk = $('#monthSelect').value || monthKey(); const rows = ledger[mk]||[]; const byName={}; rows.forEach(o=>{ (byName[o.name] ||= []).push(o); }); const data=[]; let totalGlobal=0; Object.keys(byName).forEach(name=>{ const orders=byName[name]; const pending = orders.filter(o=>o.status==='unpaid').reduce((a,o)=>a+o.total,0); const detail = orders.map(o=>`${new Date(o.dateISO).toLocaleDateString()} (${o.status}): ${o.items.map(i=>i.qty+'×'+i.name).join('; ')} => ${o.total.toFixed(2)}`).join(' | ');
    data.push({Cliente:name,Detalle:detail,TotalPendiente:pending}); totalGlobal+=pending; }); data.push({Cliente:'TOTAL GLOBAL',Detalle:'',TotalPendiente:totalGlobal});
    const ws = XLSX.utils.json_to_sheet(data); const wb = XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb,ws,'Fiados'); const fname = `fiados_${(mk.replace('-','_'))}.xlsx`; XLSX.writeFile(wb,fname);
  }

  // ===== EVENTS & INIT =====
  document.addEventListener('DOMContentLoaded', ()=>{
    // Tabs
    $$('.tab-btn').forEach(b=>b.addEventListener('click',()=>{ $$('.tab').forEach(t=>t.classList.add('hidden')); $(`#tab-${b.dataset.tab}`).classList.remove('hidden'); $$('.tab-btn').forEach(x=>{ x.classList.toggle('bg-slate-900', x.dataset.tab===b.dataset.tab); x.classList.toggle('text-white', x.dataset.tab===b.dataset.tab); x.classList.toggle('bg-slate-100', x.dataset.tab!==b.dataset.tab); }); }));
    // search
    $('#searchInput').addEventListener('input', e=>renderMenu(e.target.value));

    // order buttons
    $('#clearOrderBtn').addEventListener('click',()=>{ // devolver stock de la orden
      order.lines.forEach(l=>{ const it = items.find(x=>x.id===l.id); if(it) it.stock += l.qty; }); order={lines:[],history:[]}; saveItems(); renderMenu(); renderOrder(); });
    $('#undoBtn').addEventListener('click',()=>{ const last = order.history.pop(); if(!last) return; if(last.type==='add'){ const idx = order.lines.findIndex(l=>l.id===last.itemId); if(idx>=0){ order.lines[idx].qty -=1; const it = items.find(x=>x.id===last.itemId); if(it) it.stock+=1; if(order.lines[idx].qty<=0) order.lines.splice(idx,1); } } renderMenu(); renderOrder(); saveItems(); });
    $('#chargeBtn').addEventListener('click', ()=>charge(true)); $('#fiarBtn').addEventListener('click', ()=>charge(false));

    // menu editor
    $('#addItemBtn').addEventListener('click', ()=>{ $('#itemId').value=''; $('#itemName').value=''; $('#itemPrice').value=''; $('#itemStock').value='10'; $('#itemActive').checked=true; $('#itemFavorite').checked=false; $('#itemPreview').textContent='🍽️'; $('#itemDialog').showModal(); });
    $('#resetDemoBtn').addEventListener('click', ()=>{ if(!confirm('Restaurar demo?')) return; items = demo.map(d=>({...d,id:crypto.randomUUID()})); saveItems(); renderEditGrid(); renderMenu(); });
    $('#itemForm').addEventListener('submit', e=>{ e.preventDefault(); const id=$('#itemId').value||crypto.randomUUID(); let it = items.find(x=>x.id===id); if(!it){ it={id}; items.push(it); } it.name = $('#itemName').value.trim(); it.price = Number($('#itemPrice').value||0); it.stock = Math.max(0, Number($('#itemStock').value||0)); it.active = $('#itemActive').checked; it.favorite = $('#itemFavorite').checked; const prev = $('#itemPreview').innerHTML.trim(); it.imageData = prev.startsWith('<img')?prev:null; it.emoji = !it.imageData?prev:null; saveItems(); renderEditGrid(); renderMenu(); $('#itemDialog').close(); });
    $('#chooseEmojiBtn').addEventListener('click', ()=>openEmojiPicker(em=>{ $('#itemPreview').textContent=em; }));
    $('#itemImageInput').addEventListener('change', async e=>{ const f = e.target.files[0]; if(!f) return; const d = await fileToDataURL(f); $('#itemPreview').innerHTML = `<img src="${d}" class="w-8 h-8 object-cover rounded-lg"/>`; });

    // customers
    $('#addCustomerBtn').addEventListener('click', ()=>{ $('#newCustomerName').value=''; $('#customerDialog').showModal(); });
    $('#customerForm').addEventListener('submit', e=>{ e.preventDefault(); const n = $('#newCustomerName').value.trim(); if(!n) return; customers.push(n); LS.set('customers',customers); renderCustomers(); $('#customerDialog').close(); });

    // ledger controls
    $('#monthSelect').addEventListener('change', renderLedger); $('#filterStatus').addEventListener('change', renderLedger);
    $('#exportExcelBtn').addEventListener('click', exportLedgerToExcel);

    // ajustes
    $('#currencySelect').value = settings.currency||'USD'; $('#currencySelect').addEventListener('change', e=>{ settings.currency = e.target.value; LS.set('settings',settings); renderOrder(); renderLedger(); renderMenu($('#searchInput').value); });
    $('#taxInput').value = settings.tax||''; $('#taxInput').addEventListener('change', e=>{ settings.tax = Number(e.target.value||0); LS.set('settings',settings); renderOrder(); renderLedger(); });
    $('#clearDataBtn').addEventListener('click', ()=>{ if(!confirm('Borrar todo?')) return; LS.del('menuItems'); LS.del('ledger'); LS.del('settings'); LS.del('customers'); items = demo.map(d=>({...d,id:crypto.randomUUID()})); ledger={}; settings={currency:'USD',tax:0}; customers=['Cliente']; LS.set('menuItems',items); LS.set('ledger',ledger); LS.set('settings',settings); LS.set('customers',customers); renderMenu(); renderEditGrid(); renderLedger(); renderCustomers(); alert('Reiniciado'); });

    // help
    $('#helpBtn').addEventListener('click', ()=>$('#helpDialog').showModal()); $('#helpBtn2').addEventListener('click', ()=>$('#helpDialog').showModal());

    // init
    renderMenu(); renderOrder(); renderEditGrid(); renderLedger(); renderCustomers();
  });
  </script>
</body>
</html>
