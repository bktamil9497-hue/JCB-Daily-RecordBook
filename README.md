import os, zipfile, textwrap
outdir = "/mnt/data/JCB_Daily_Record_Book"
os.makedirs(outdir, exist_ok=True)
html = r'''<!doctype html>
<html lang="ta">
<head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<meta name="theme-color" content="#07345a">
<title>JCB Daily Record Book</title>
<style>
:root{--navy:#07345a;--gold:#f9b900;--bg:#f3f6fa;--green:#137653}
*{box-sizing:border-box}body{margin:0;font-family:system-ui,-apple-system,"Noto Sans Tamil",sans-serif;background:var(--bg);color:#182433}
header{background:var(--navy);color:white;padding:18px 16px;position:sticky;top:0;z-index:2;box-shadow:0 2px 8px #0002}
header h1{font-size:21px;margin:0}header p{margin:4px 0 0;font-size:12px;opacity:.9}
main{max-width:760px;margin:auto;padding:12px}.card{background:white;border-radius:14px;padding:15px;margin:12px 0;box-shadow:0 2px 10px #132b4310}
h2{font-size:17px;margin:0 0 12px;color:var(--navy)}label{display:block;font-size:13px;color:#4c5a69;margin:10px 0 5px}
input,textarea,select{font:inherit;width:100%;border:1px solid #cbd5df;border-radius:9px;padding:11px;background:white;color:#152333}
.grid{display:grid;grid-template-columns:1fr 1fr;gap:10px}.grid .full{grid-column:1/-1}
.results{display:grid;grid-template-columns:repeat(3,1fr);gap:8px}.metric{background:#edf5f1;border-radius:10px;padding:10px}.metric small{display:block;color:#54665e;font-size:11px}.metric strong{display:block;color:var(--green);font-size:16px;overflow-wrap:anywhere}
button{border:0;border-radius:10px;padding:12px 14px;font:inherit;font-weight:700;cursor:pointer}
.primary{background:var(--gold);color:#152333;width:100%;margin-top:10px}.secondary{background:#e6edf5;color:var(--navy)}.danger{background:#fde9e7;color:#9a2b20}.actions{display:flex;gap:8px;flex-wrap:wrap}.actions button{flex:1}
.entry{border:1px solid #e1e7ee;border-radius:10px;padding:12px;margin:9px 0}.entry h3{margin:0 0 5px;font-size:15px}.entry p{margin:4px 0;font-size:13px;color:#4d5a68}.entry img{width:100%;max-height:190px;object-fit:cover;border-radius:8px;margin-top:8px}
.note{font-size:12px;color:#627083;line-height:1.5}.hidden{display:none!important}
@media(max-width:420px){.grid{grid-template-columns:1fr 1fr}.results{grid-template-columns:1fr 1fr}.results .metric:last-child{grid-column:1/-1}}
</style></head>
<body>
<header><h1>🚜 JCB Daily Record Book</h1><p>வேலை பதிவு • தானியங்கி கணக்கு • மாத அறிக்கை</p></header>
<main>
<section class="card"><h2>புதிய வேலை பதிவு / New Entry</h2>
<div class="grid">
<div><label>தேதி / Date</label><input id="date" type="date"></div>
<div><label>வேலை பெயர் / Customer</label><input id="name" placeholder="Paper company"></div>
<div class="full"><label>இடம் / Location</label><input id="location" placeholder="வேலை செய்த இடம்"></div>
<div><label>Starting Reading</label><input id="start" type="number" step="0.1" inputmode="decimal" placeholder="1425.9"></div>
<div><label>Closing Reading</label><input id="close" type="number" step="0.1" inputmode="decimal" placeholder="1434.3"></div>
<div><label>Rate per hour (₹)</label><input id="rate" type="number" step="0.01" inputmode="decimal" placeholder="இந்த வேலைக்கான rate"></div>
<div><label>Diesel Liter</label><input id="dieselL" type="number" step="0.1" inputmode="decimal" value="0"></div>
<div><label>Diesel Amount (₹)</label><input id="diesel" type="number" step="0.01" inputmode="decimal" value="0"></div>
<div><label>Driver Bata (₹)</label><input id="bata" type="number" step="0.01" inputmode="decimal" value="0"></div>
<div><label>Advance (₹)</label><input id="advance" type="number" step="0.01" inputmode="decimal" value="0"></div>
<div><label>Food (₹)</label><input id="food" type="number" step="0.01" inputmode="decimal" value="0"></div>
<div><label>Salary / Other (₹)</label><input id="salary" type="number" step="0.01" inputmode="decimal" value="0"></div>
<div class="full"><label>JCB Photo</label><input id="photo" type="file" accept="image/*" capture="environment"><p class="note">படம் இந்த பதிவுடன் சேமிக்கப்படும்; பெரிய படங்கள் மொபைல் storage-ஐ அதிகம் பயன்படுத்தலாம்.</p></div>
<div class="full"><label>கூடுதல் விவரம் / Notes</label><textarea id="notes" rows="2" placeholder="விருப்பப்படி குறிப்புகள்"></textarea></div>
</div>
<div class="results" style="margin-top:12px">
<div class="metric"><small>Total Hours</small><strong id="hours">0.00</strong></div>
<div class="metric"><small>Total Amount</small><strong id="amount">₹0.00</strong></div>
<div class="metric"><small>Balance</small><strong id="balance">₹0.00</strong></div>
</div>
<div id="customFields"></div>
<div class="actions" style="margin-top:12px"><button class="secondary" onclick="addCustomField()">＋ புதிய Field</button><button class="secondary" onclick="clearForm()">Clear</button></div>
<button class="primary" onclick="saveEntry()">Save Entry / பதிவு சேமி</button>
<p class="note">கணக்கு: Hours = Closing − Starting. Amount = Hours × Rate. Balance = Amount − Diesel − Bata − Advance − Food − Salary. உங்களுடைய கணக்கு முறை வேறாக இருந்தால் மாற்றிக்கொள்ளலாம்.</p>
</section>
<section class="card"><h2>பதிவுகள் / Saved Records</h2>
<div class="grid"><div><label>தேதி/மாதம் தேடல்</label><input id="filter" type="month"></div><div style="align-self:end"><button class="secondary" style="width:100%" onclick="render()">Filter</button></div></div>
<div class="results" style="margin:12px 0"><div class="metric"><small>Entries</small><strong id="count">0</strong></div><div class="metric"><small>Total Hours</small><strong id="sumHours">0.00</strong></div><div class="metric"><small>Total Amount</small><strong id="sumAmount">₹0.00</strong></div></div>
<div class="actions"><button class="secondary" onclick="exportCSV()">Export CSV</button><button class="secondary" onclick="shareBackup()">Backup JSON</button></div>
<div id="records"></div>
</section>
<p class="note">தரவு இந்த browser-இல் உள்ளூராகச் சேமிக்கப்படும். Browser data clear செய்தால் பதிவுகள் அழியலாம்; அவ்வப்போது Backup JSON / Export CSV எடுக்கவும். இது முதல் mobile-ready version; APK அல்ல.</p>
</main>
<script>
const $=id=>document.getElementById(id);
const fields=['date','name','location','start','close','rate','dieselL','diesel','bata','advance','food','salary','notes'];
let photoData='', editingId=null;
const num=id=>parseFloat($(id).value)||0;
const money=n=>'₹'+Number(n||0).toLocaleString('en-IN',{minimumFractionDigits:2,maximumFractionDigits:2});
function calc(){let h=Math.max(0,num('close')-num('start')), a=h*num('rate'), b=a-num('diesel')-num('bata')-num('advance')-num('food')-num('salary');$('hours').textContent=h.toFixed(2);$('amount').textContent=money(a);$('balance').textContent=money(b);return {hours:h,amount:a,balance:b}}
['start','close','rate','diesel','bata','advance','food','salary'].forEach(id=>$(id).addEventListener('input',calc));
$('photo').addEventListener('change',e=>{const f=e.target.files[0];if(!f){photoData='';return}const r=new FileReader();r.onload=()=>photoData=r.result;r.readAsDataURL(f)});
function getEntries(){try{return JSON.parse(localStorage.getItem('jcb_entries')||'[]')}catch(e){return []}}
function setEntries(a){localStorage.setItem('jcb_entries',JSON.stringify(a))}
function addCustomField(){const wrap=document.createElement('div');wrap.className='grid';wrap.style.marginTop='8px';wrap.innerHTML='<div><label>Field பெயர்</label><input class="cfname" placeholder="Tool charges"></div><div><label>Value</label><input class="cfvalue" placeholder="தொகை அல்லது விவரம்"></div>'; $('customFields').appendChild(wrap)}
function clearForm(){fields.forEach(id=>{if(id==='date')$(id).value=new Date().toISOString().slice(0,10);else if(['dieselL','diesel','bata','advance','food','salary'].includes(id))$(id).value='0';else $(id).value=''});$('photo').value='';photoData='';$('customFields').innerHTML='';editingId=null;calc()}
function saveEntry(){if(!$('start').value||!$('close').value){alert('Starting மற்றும் Closing Reading நிரப்பவும்.');return}const c=calc(), custom=[...document.querySelectorAll('#customFields .cfname')].map((el,i)=>({name:el.value,value:document.querySelectorAll('#customFields .cfvalue')[i].value})).filter(x=>x.name);const entry={id:editingId||Date.now(),date:$('date').value,name:$('name').value,location:$('location').value,start:num('start'),close:num('close'),rate:num('rate'),dieselL:num('dieselL'),diesel:num('diesel'),bata:num('bata'),advance:num('advance'),food:num('food'),salary:num('salary'),notes:$('notes').value,photo:photoData,hours:c.hours,amount:c.amount,balance:c.balance,custom};let a=getEntries();if(editingId)a=a.map(x=>x.id===editingId?entry:x);else a.unshift(entry);setEntries(a);alert('பதிவு சேமிக்கப்பட்டது!');clearForm();render()}
function render(){let all=getEntries(),filter=$('filter').value;if(filter)all=all.filter(e=>(e.date||'').startsWith(filter));$('count').textContent=all.length;$('sumHours').textContent=all.reduce((s,e)=>s+(+e.hours||0),0).toFixed(2);$('sumAmount').textContent=money(all.reduce((s,e)=>s+(+e.amount||0),0));$('records').innerHTML='';if(!all.length){$('records').innerHTML='<p class="note">இன்னும் பதிவு இல்லை.</p>';return}all.forEach(e=>{let div=document.createElement('div');div.className='entry';let custom=(e.custom||[]).map(x=>`<p>${esc(x.name)}: ${esc(x.value)}</p>`).join('');div.innerHTML=`<h3>${esc(e.date||'')} • ${esc(e.name||'வேலை பெயர் இல்லை')}</h3><p>${esc(e.location||'')}</p><p>Reading: ${e.start} → ${e.close} | Hours: ${(+e.hours).toFixed(2)}</p><p>Rate: ${money(e.rate)} | Amount: <b>${money(e.amount)}</b></p><p>Diesel: ${money(e.diesel)} | Bata: ${money(e.bata)} | Advance: ${money(e.advance)} | Food: ${money(e.food)} | Salary: ${money(e.salary)}</p><p><b>Balance: ${money(e.balance)}</b></p><p>${esc(e.notes||'')}</p>${custom}${e.photo?`<img alt="JCB photo" src="${e.photo}">`:''}<div class="actions" style="margin-top:8px"><button class="secondary" data-edit="${e.id}">Edit</button><button class="danger" data-delete="${e.id}">Delete</button></div>`;$('records').appendChild(div)});document.querySelectorAll('[data-delete]').forEach(b=>b.onclick=()=>{if(confirm('இந்த பதிவை நீக்கவா?')){setEntries(getEntries().filter(e=>e.id!=b.dataset.delete));render()}});document.querySelectorAll('[data-edit]').forEach(b=>b.onclick=()=>editEntry(+b.dataset.edit))}
function editEntry(id){const e=getEntries().find(x=>x.id===id);if(!e)return;fields.forEach(k=>{if($(k))$(k).value=e[k]??''});photoData=e.photo||'';$('customFields').innerHTML='';(e.custom||[]).forEach(x=>{addCustomField();let i=document.querySelectorAll('.cfname').length-1;document.querySelectorAll('.cfname')[i].value=x.name;document.querySelectorAll('.cfvalue')[i].value=x.value});editingId=id;calc();window.scrollTo({top:0,behavior:'smooth'})}
function esc(s){return String(s??'').replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]))}
function exportCSV(){const a=getEntries();const keys=['date','name','location','start','close','hours','rate','amount','dieselL','diesel','bata','advance','food','salary','balance','notes'];const q=v=>'"'+String(v??'').replace(/"/g,'""')+'"';let csv=[keys.join(','),...a.map(e=>keys.map(k=>q(e[k])).join(','))].join('\r\n');download(new Blob(['\ufeff'+csv],{type:'text/csv;charset=utf-8'}),'jcb-records.csv')}
function shareBackup(){download(new Blob([JSON.stringify(getEntries(),null,2)],{type:'application/json'}),'jcb-backup.json')}
function download(blob,name){const url=URL.createObjectURL(blob),a=document.createElement('a');a.href=url;a.download=name;a.click();setTimeout(()=>URL.revokeObjectURL(url),1000)}
$('date').value=new Date().toISOString().slice(0,10);$('filter').value=new Date().toISOString().slice(0,7);calc();render();
</script></body></html>'''
html_path=os.path.join(outdir,"index.html")
with open(html_path,"w",encoding="utf-8") as f:f.write(html)
zip_path="/mnt/data/JCB_Daily_Record_Book_Mobile.zip"
with zipfile.ZipFile(zip_path,"w",zipfile.ZIP_DEFLATED) as z:
    z.write(html_path,"JCB_Daily_Record_Book/index.html")
print("Created mobile-ready JCB record book app files.")
