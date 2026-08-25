# Jobbin safety pass — exact anchored edits

Apply to the current `index.html` from commit `a84e2b4`. Use Copilot Chat in Agent mode. Review the diff before accepting. Do not touch any other `<script>` tag.

Parent milestone for the commit: `#869e3bz4y`

## 1. Add linked-record protection and rich-text cleaning

Ctrl+F this unique block:

```js
function jobSupplier(w){ var p=(w&&w.supplierId)?scpById(w.supplierId):null; return p?p.company:((w&&w.supplier)||''); }
function rowIdOf(el){ var tr=el.closest('tr'); return tr&&tr.dataset.id; }
function cardIdOf(el){ var c=el.closest('.pcard'); return c&&c.dataset.id; }
var esc=function(s){ return (s==null?'':String(s)).replace(/[&<>"']/g,function(c){return{'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]; }); };
```

Replace it with:

```js
function jobSupplier(w){ var p=(w&&w.supplierId)?scpById(w.supplierId):null; return p?p.company:((w&&w.supplier)||''); }
function jobsUsingSiteIds(ids){ var set={}; (ids||[]).forEach(function(id){ if(id)set[id]=1; }); return (DB.works||[]).filter(function(w){ return !!set[w.siteId]; }).length; }
function jobsUsingClient(c){ var seen={}; return (DB.works||[]).filter(function(w){ if(w.clientId===c.id){seen[w.id]=1;return true;} return false; }).length + jobsUsingSiteIds(sitesUnder(c).map(function(s){return s.id;})); }
function jobsUsingManager(m){ return jobsUsingSiteIds((m.sites||[]).map(function(s){return s.id;})); }
function jobsUsingSite(s){ return jobsUsingSiteIds([s.id]); }
function jobsUsingScp(s){ return (DB.works||[]).filter(function(w){ return w.supplierId===s.id; }).length; }
function blockLinkedDelete(label,count){
  if(!count)return false;
  toast(label+' is linked to '+count+' job'+(count===1?'':'s')+'. Set it to Stopped instead.');
  return true;
}
function rowIdOf(el){ var tr=el.closest('tr'); return tr&&tr.dataset.id; }
function cardIdOf(el){ var c=el.closest('.pcard'); return c&&c.dataset.id; }
var esc=function(s){ return (s==null?'':String(s)).replace(/[&<>"']/g,function(c){return{'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]; }); };
function cleanRichHtml(html){
  var t=document.createElement('template');
  t.innerHTML=String(html||'');
  var allowed={B:1,STRONG:1,P:1,BR:1,UL:1,OL:1,LI:1,DIV:1};
  var blocked={SCRIPT:1,STYLE:1,IFRAME:1,OBJECT:1,EMBED:1,SVG:1,MATH:1};
  Array.prototype.slice.call(t.content.querySelectorAll('*')).reverse().forEach(function(el){
    if(blocked[el.tagName]){ el.remove(); return; }
    if(!allowed[el.tagName]){ el.replaceWith.apply(el,Array.prototype.slice.call(el.childNodes)); return; }
    Array.prototype.slice.call(el.attributes).forEach(function(a){ el.removeAttribute(a.name); });
  });
  return t.innerHTML;
}
```

## 2. Make migrations fail safely, validate backups, and queue saves

Ctrl+F this unique block:

```js
function runMigrations(data){
  if(!data||typeof data!=='object')return data;
  var v=data.schemaVersion||1;
  while(v<CURRENT_SCHEMA_VERSION&&MIGRATIONS[v-1]){ data=MIGRATIONS[v-1](data); v++; }
  data.schemaVersion=CURRENT_SCHEMA_VERSION;
  return data;
}
var DB;
function save(){
  try{ idbPutState(DB).catch(function(){ toast('Could not save to device storage'); }); }catch(e){}
  var sm=document.getElementById('storageMeterWrap'); if(sm) renderStorageMeter(sm);
  driveAutoSync();
  return true;
}
```

Replace it with:

```js
function runMigrations(data){
  if(!data||typeof data!=='object')return data;
  var v=data.schemaVersion||1;
  while(v<CURRENT_SCHEMA_VERSION){
    if(typeof MIGRATIONS[v-1]!=='function') throw new Error('Missing data upgrade from schema '+v);
    data=MIGRATIONS[v-1](data); v++;
  }
  data.schemaVersion=CURRENT_SCHEMA_VERSION;
  return data;
}
function prepareRegister(raw){
  var data=(raw&&raw.data)?raw.data:raw;
  if(!data||typeof data!=='object'||Array.isArray(data)) throw new Error('Backup is not a Jobbin register');
  ['works','internal','clients','scps','tasks','trades'].forEach(function(k){
    if(k in data&&!Array.isArray(data[k])) throw new Error('Backup field '+k+' is invalid');
  });
  var next=runMigrations(Object.assign(blank(),data));
  (next.works||[]).forEach(function(w){
    if(!w||typeof w!=='object') throw new Error('Backup contains an invalid job');
    if(!Array.isArray(w.comments))w.comments=[];
    if(!Array.isArray(w.events))w.events=[];
    w.comments.forEach(function(c){ if(c&&c.html)c.html=cleanRichHtml(c.html); });
  });
  (next.clients||[]).forEach(function(c){
    if(!c||typeof c!=='object') throw new Error('Backup contains an invalid client');
    if(!Array.isArray(c.managers))c.managers=[];
    c.managers.forEach(function(m){ if(!Array.isArray(m.sites))m.sites=[]; m.sites.forEach(function(s){if(!Array.isArray(s.people))s.people=[];}); });
  });
  (next.scps||[]).forEach(function(s){
    if(!s||typeof s!=='object')throw new Error('Backup contains an invalid contractor');
    if(!Array.isArray(s.trades))s.trades=[];
    if(!Array.isArray(s.people))s.people=[];
  });
  return next;
}
var DB, SAVE_CHAIN=Promise.resolve(), SAVE_PENDING=0;
function save(){
  var snapshot;
  try{ snapshot=(typeof structuredClone==='function')?structuredClone(DB):JSON.parse(JSON.stringify(DB)); }
  catch(e){ snapshot=DB; }
  SAVE_PENDING++;
  var write=SAVE_CHAIN.catch(function(){}).then(function(){ return idbPutState(snapshot); });
  SAVE_CHAIN=write.then(function(){SAVE_PENDING--;},function(){SAVE_PENDING--;throw new Error('save failed');});
  write.catch(function(){ showToast('Could not save to device storage'); });
  var sm=document.getElementById('storageMeterWrap'); if(sm) renderStorageMeter(sm);
  driveAutoSync();
  return write;
}
```

## 3. Make toast messages wait for storage

Ctrl+F this unique whole line:

```js
function toast(msg){ var w=$('#toasts'); if(!w)return; var t=document.createElement('div'); t.className='toast'; t.innerHTML='<i data-lucide="circle-check-big"></i>'+esc(msg); w.appendChild(t); icons(); setTimeout(function(){t.style.opacity='0';t.style.transition='opacity .3s';setTimeout(function(){t.remove();},300);},2600); }
```

Replace it with:

```js
function showToast(msg){ var w=$('#toasts'); if(!w)return; var t=document.createElement('div'); t.className='toast'; t.innerHTML='<i data-lucide="circle-check-big"></i>'+esc(msg); w.appendChild(t); icons(); setTimeout(function(){t.style.opacity='0';t.style.transition='opacity .3s';setTimeout(function(){t.remove();},300);},2600); }
function toast(msg){ if(SAVE_PENDING){ SAVE_CHAIN.then(function(){showToast(msg);}).catch(function(){}); return; } showToast(msg); }
```

## 4. Record app and schema versions separately in Drive backups

Ctrl+F:

```js
  var payload=JSON.stringify({app:'jobbin',v:'31.4',savedAt:Date.now(),data:DB});
```

Replace with:

```js
  var payload=JSON.stringify({app:'jobbin',appVersion:'31.4',schemaVersion:CURRENT_SCHEMA_VERSION,savedAt:Date.now(),data:DB});
```

## 5. Validate and persist a Drive restore before switching data

Ctrl+F this full function:

```js
function drivePull(){
  if(drive.busy)return Promise.resolve(); drive.busy=true; driveStatus('Restoring from Drive…');
  return driveFind().then(function(id){ if(!id) throw new Error('No Drive backup found'); return driveApi('https://www.googleapis.com/drive/v3/files/'+id+'?alt=media'); })
    .then(function(r){return r.json();})
    .then(function(obj){ var data=obj&&obj.data?obj.data:obj; DB=runMigrations(Object.assign(blank(),data)); var cfg=driveCfg(); cfg.lastSync=cfg.lastPull=Date.now(); save(); applyTheme(); driveStatus('Restored from Drive','ok'); drive.busy=false; go('dashboard'); toast('Restored from Google Drive'); })
    .catch(function(e){ drive.busy=false; driveStatus('Restore failed: '+(e.message||e),'err'); });
}
```

Replace with:

```js
function drivePull(){
  if(drive.busy)return Promise.resolve(); drive.busy=true; driveStatus('Restoring from Drive…');
  return driveFind().then(function(id){ if(!id) throw new Error('No Drive backup found'); return driveApi('https://www.googleapis.com/drive/v3/files/'+id+'?alt=media'); })
    .then(function(r){return r.json();})
    .then(function(obj){
      var next=prepareRegister(obj);
      return idbPutState(next).then(function(){
        DB=next;
        var cfg=driveCfg(); cfg.lastSync=cfg.lastPull=Date.now();
        return idbPutState(DB);
      });
    })
    .then(function(){ applyTheme(); driveStatus('Restored from Drive','ok'); drive.busy=false; go('dashboard'); showToast('Restored from Google Drive'); })
    .catch(function(e){ drive.busy=false; driveStatus('Restore failed: '+(e.message||e),'err'); });
}
```

## 6. Stop Excel treating exported data as formulas

Ctrl+F:

```js
    var qq=function(v){ v=(v==null?'':String(v)); return '"'+v.replace(/"/g,'""')+'"'; };
```

Replace with:

```js
    var qq=function(v){ v=(v==null?'':String(v)); if(/^[=+\-@\t\r]/.test(v))v="'"+v; return '"'+v.replace(/"/g,'""')+'"'; };
```

## 7. Validate and persist imports before switching data

Ctrl+F:

```js
function importBackup(e){
  var f=e.target.files&&e.target.files[0]; if(!f)return;
  var rd=new FileReader();
  rd.onload=function(){ try{ var obj=JSON.parse(rd.result); DB=runMigrations(Object.assign(blank(),obj)); save(); applyTheme(); go('dashboard'); toast('Backup imported'); }catch(err){ toast('That file is not a valid backup'); } };
  rd.readAsText(f);
}
```

Replace with:

```js
function importBackup(e){
  var f=e.target.files&&e.target.files[0]; if(!f)return;
  var rd=new FileReader();
  rd.onload=function(){
    var next;
    try{ next=prepareRegister(JSON.parse(rd.result)); }
    catch(err){ toast('That file is not a valid Jobbin backup'); return; }
    idbPutState(next).then(function(){ DB=next; applyTheme(); go('dashboard'); showToast('Backup imported'); })
      .catch(function(){ toast('Import failed. Your current data is unchanged.'); });
  };
  rd.readAsText(f);
}
```

## 8. Clean rich comments on save, edit and render

Ctrl+F:

```js
  r.comments.push({id:uid(),t:Date.now(),by:'L. Smith',type:type||'note',html:html||'',txt:text,attachments:files||[]});
```

Replace with:

```js
  r.comments.push({id:uid(),t:Date.now(),by:'L. Smith',type:type||'note',html:cleanRichHtml(html||''),txt:text,attachments:files||[]});
```

Ctrl+F:

```js
      var body=c.html?c.html:esc(c.txt||'');
```

Replace with:

```js
      var body=c.html?cleanRichHtml(c.html):esc(c.txt||'');
```

Ctrl+F this whole line:

```js
  var commit=function(){ var html=ce.innerHTML.trim(), text=(ce.textContent||'').trim(); if(text){ c.html=html; c.txt=text; c.edited=true; save(); } refreshAll(); refreshOverviewRow(r.id); };
```

Replace with:

```js
  var commit=function(){ var html=cleanRichHtml(ce.innerHTML.trim()), text=(ce.textContent||'').trim(); if(text){ c.html=html; c.txt=text; c.edited=true; save().then(function(){toast('Update saved');}); } refreshAll(); refreshOverviewRow(r.id); };
```

## 9. Block deletion of linked clients, managers, sites and contractors

Ctrl+F the exact client-delete line:

```js
      if((b=e.target.closest('[data-delclient]'))){ var id=b.getAttribute('data-delclient'),cc=clientById(id); confirmDel('Delete client?',cc.name+' and all its managers, sites & people will be removed.',function(){ DB.clients=DB.clients.filter(function(x){return x.id!==id;}); save(); render(); toast('Client deleted'); }); return; }
```

Replace with:

```js
      if((b=e.target.closest('[data-delclient]'))){ var id=b.getAttribute('data-delclient'),cc=clientById(id); if(blockLinkedDelete(cc.name,jobsUsingClient(cc)))return; confirmDel('Delete client?',cc.name+' and all its managers, sites & people will be removed.',function(){ DB.clients=DB.clients.filter(function(x){return x.id!==id;}); save(); render(); toast('Client deleted'); }); return; }
```

Ctrl+F the exact manager-delete line:

```js
      if((b=e.target.closest('[data-delmgr]'))){ var p2=b.getAttribute('data-delmgr').split(':'),cx=clientById(p2[0]),mx=byId(cx.managers,p2[1]); confirmDel('Delete manager?',mx.name+' and their sites will be removed.',function(){ cx.managers=cx.managers.filter(function(x){return x.id!==p2[1];}); save(); render(); toast('Manager deleted'); }); return; }
```

Replace with:

```js
      if((b=e.target.closest('[data-delmgr]'))){ var p2=b.getAttribute('data-delmgr').split(':'),cx=clientById(p2[0]),mx=byId(cx.managers,p2[1]); if(blockLinkedDelete(mx.name,jobsUsingManager(mx)))return; confirmDel('Delete manager?',mx.name+' and their sites will be removed.',function(){ cx.managers=cx.managers.filter(function(x){return x.id!==p2[1];}); save(); render(); toast('Manager deleted'); }); return; }
```

Ctrl+F the exact site-delete line:

```js
      if((b=e.target.closest('[data-delsite]'))){ var p5=b.getAttribute('data-delsite').split(':'),cy=clientById(p5[0]),my=byId(cy.managers,p5[1]),sy=byId(my.sites,p5[2]); confirmDel('Delete site?',sy.name+' will be removed.',function(){ my.sites=my.sites.filter(function(x){return x.id!==p5[2];}); save(); render(); toast('Site deleted'); }); return; }
```

Replace with:

```js
      if((b=e.target.closest('[data-delsite]'))){ var p5=b.getAttribute('data-delsite').split(':'),cy=clientById(p5[0]),my=byId(cy.managers,p5[1]),sy=byId(my.sites,p5[2]); if(blockLinkedDelete(sy.name,jobsUsingSite(sy)))return; confirmDel('Delete site?',sy.name+' will be removed.',function(){ my.sites=my.sites.filter(function(x){return x.id!==p5[2];}); save(); render(); toast('Site deleted'); }); return; }
```

There are TWO SCP delete routes. Replace both.

Ctrl+F:

```js
      if((b=e.target.closest('[data-delscp]'))){ var id=cardIdOf(e.target),s=scpById(id); confirmDel('Delete SCP?',s.company+' will be removed.',function(){ DB.scps=DB.scps.filter(function(x){return x.id!==id;}); save(); render(); toast('SCP deleted'); }); return; }
```

Replace with:

```js
      if((b=e.target.closest('[data-delscp]'))){ var id=cardIdOf(e.target),s=scpById(id); if(blockLinkedDelete(s.company,jobsUsingScp(s)))return; confirmDel('Delete SCP?',s.company+' will be removed.',function(){ DB.scps=DB.scps.filter(function(x){return x.id!==id;}); save(); render(); toast('SCP deleted'); }); return; }
```

Ctrl+F:

```js
  if(id){ $('#scpDel').addEventListener('click',function(){ confirmDel('Delete SCP?',s.company+' will be removed.',function(){ DB.scps=DB.scps.filter(function(x){return x.id!==id;}); save(); close(); render(); toast('SCP deleted'); }); }); }
```

Replace with:

```js
  if(id){ $('#scpDel').addEventListener('click',function(){ if(blockLinkedDelete(s.company,jobsUsingScp(s)))return; confirmDel('Delete SCP?',s.company+' will be removed.',function(){ DB.scps=DB.scps.filter(function(x){return x.id!==id;}); save(); close(); render(); toast('SCP deleted'); }); }); }
```

## 10. Verification before commit

1. Ctrl+F each original anchor again. Every old block must return zero matches.
2. Ctrl+F `cleanRichHtml(`. It should appear in the function definition, backup preparation, add-comment, timeline render and comment edit.
3. Ctrl+F `blockLinkedDelete(`. It should appear in the helper plus client, manager, site and both SCP delete routes.
4. Run the acceptance tests on the ClickUp task.
5. Commit message: `Close five data-safety holes found in full-file audit #869e3bz4y`
6. Sync, wait for Cloudflare, then re-read the public raw `index.html` and verify the anchors landed.
