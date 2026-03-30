<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no"/>
  <meta name="theme-color" content="#0b1120"/>
  <meta name="apple-mobile-web-app-capable" content="yes"/>
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent"/>
  <meta name="mobile-web-app-capable" content="yes"/>
  <title>Cricket Toss Book</title>
  <link rel="manifest" href="data:application/json,{%22name%22:%22Cricket+Toss+Book%22,%22short_name%22:%22TossBook%22,%22start_url%22:%22.%22,%22display%22:%22standalone%22,%22background_color%22:%22%230b1120%22,%22theme_color%22:%22%230b1120%22}"/>

  <!-- React + ReactDOM UMD -->
  <script crossorigin src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
  <script crossorigin src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
  <!-- Babel standalone for JSX -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.5/babel.min.js"></script>

  <style>
    /* Splash / loading screen */
    #splash{position:fixed;inset:0;background:#0b1120;display:flex;flex-direction:column;align-items:center;justify-content:center;z-index:9999;transition:opacity .4s ease}
    #splash.hidden{opacity:0;pointer-events:none}
    .splash-icon{font-size:52px;margin-bottom:12px;animation:splashPop .6s cubic-bezier(.34,1.56,.64,1) both}
    .splash-title{font-family:system-ui,sans-serif;font-size:22px;font-weight:800;color:#f59e0b;letter-spacing:-.5px;margin-bottom:6px}
    .splash-sub{font-family:system-ui,sans-serif;font-size:13px;color:#64748b}
    .splash-loader{width:40px;height:3px;background:#1f2d45;border-radius:3px;margin-top:28px;overflow:hidden}
    .splash-bar{height:100%;width:0%;background:#f59e0b;border-radius:3px;animation:loadBar 1s ease forwards .3s}
    @keyframes splashPop{0%{transform:scale(.4);opacity:0}70%{transform:scale(1.1)}100%{transform:scale(1);opacity:1}}
    @keyframes loadBar{0%{width:0%}100%{width:100%}}
    #root{min-height:100vh}
  </style>
</head>
<body>

<!-- Splash screen -->
<div id="splash">
  <div class="splash-icon">🏏</div>
  <div class="splash-title">Cricket Toss Book</div>
  <div class="splash-sub">The Ultimate Toss Prediction Platform</div>
  <div class="splash-loader"><div class="splash-bar"></div></div>
</div>

<div id="root"></div>

<script type="text/babel" data-presets="react">
// ── Make React hooks available globally (since no imports) ──
const { useState, useEffect, useCallback, useMemo } = React;


/* ─────────────────────────────────────────────────────────
   STYLES
───────────────────────────────────────────────────────── */
const CSS = `
@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&family=Syne:wght@700;800&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
html{-webkit-text-size-adjust:100%}
:root{
  --bg:#0b1120;--c1:#111827;--c2:#1a2236;--c3:#1f2d45;
  --br:rgba(255,255,255,0.07);
  --gold:#f59e0b;--gold2:#d97706;--goldT:rgba(245,158,11,.12);
  --gr:#10b981;--grT:rgba(16,185,129,.12);
  --rd:#f43f5e;--rdT:rgba(244,63,94,.12);
  --bl:#3b82f6;--blT:rgba(59,130,246,.12);
  --tg:#0ea5e9;--tgT:rgba(14,165,233,.15);
  --tx:#e2e8f0;--mu:#64748b;--mu2:#94a3b8;
  --r8:8px;--r12:12px;--r16:16px;
  --nav-h:60px;
}
body{font-family:'Plus Jakarta Sans',sans-serif;background:var(--bg);color:var(--tx);min-height:100vh;font-size:14px;line-height:1.5;overflow-x:hidden}
::-webkit-scrollbar{width:3px;height:3px}::-webkit-scrollbar-thumb{background:var(--c3);border-radius:3px}
@keyframes fadeUp{from{opacity:0;transform:translateY(12px)}to{opacity:1;transform:translateY(0)}}
@keyframes popIn{0%{transform:scale(.84);opacity:0}65%{transform:scale(1.03)}100%{transform:scale(1);opacity:1}}
@keyframes slideIn{from{transform:translateX(110%);opacity:0}to{transform:translateX(0);opacity:1}}
@keyframes fadeIn{from{opacity:0}to{opacity:1}}
@keyframes pulse{0%,100%{opacity:1}50%{opacity:.35}}
.fu{animation:fadeUp .32s ease both}
.pi{animation:popIn .3s cubic-bezier(.34,1.56,.64,1) both}
/* inputs */
input,select,textarea{width:100%;padding:10px 14px;background:var(--c2);border:1px solid var(--br);border-radius:var(--r8);color:var(--tx);font-family:inherit;font-size:16px;outline:none;transition:.18s}
input:focus,select:focus{border-color:var(--gold);box-shadow:0 0 0 3px rgba(245,158,11,.1)}
input::placeholder{color:var(--mu)}input:disabled{opacity:.45;cursor:not-allowed}
select{cursor:pointer;appearance:none;background-image:url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='10' height='6' fill='%2364748b'%3E%3Cpath d='M0 0l5 6 5-6z'/%3E%3C/svg%3E");background-repeat:no-repeat;background-position:right 12px center}
/* buttons */
button{font-family:inherit;cursor:pointer;border:none;transition:.16s;display:inline-flex;align-items:center;justify-content:center;gap:6px;font-weight:600;-webkit-tap-highlight-color:transparent}
button:active{transform:scale(.96)}button:disabled{opacity:.45;pointer-events:none}
.btn{padding:11px 18px;border-radius:var(--r8);font-size:14px}
.btn-gold{background:var(--gold);color:#000}.btn-gold:hover{background:var(--gold2)}
.btn-gr{background:var(--gr);color:#fff}
.btn-rd{background:var(--rd);color:#fff}
.btn-bl{background:var(--bl);color:#fff}
.btn-tg{background:linear-gradient(135deg,#0088cc,#006fa8);color:#fff}
.btn-ghost{background:transparent;color:var(--mu2);border:1px solid var(--br)}.btn-ghost:hover{color:var(--tx);border-color:rgba(255,255,255,.18)}
.btn-sm{padding:7px 13px;font-size:13px;border-radius:6px}
.btn-lg{padding:14px 24px;font-size:15px;border-radius:var(--r12)}
.btn-w{width:100%}
/* cards */
.card{background:var(--c1);border:1px solid var(--br);border-radius:var(--r16)}
.card2{background:var(--c2);border:1px solid var(--br);border-radius:var(--r12)}
/* badges */
.badge{display:inline-flex;align-items:center;gap:3px;padding:2px 9px;border-radius:20px;font-size:11.5px;font-weight:700;white-space:nowrap}
.bg{background:var(--grT);color:var(--gr)}.br{background:var(--rdT);color:var(--rd)}
.bb{background:var(--blT);color:var(--bl)}.bgo{background:var(--goldT);color:var(--gold)}
.bm{background:rgba(100,116,139,.15);color:var(--mu2)}.bp{background:rgba(139,92,246,.15);color:#a78bfa}
/* table */
table{width:100%;border-collapse:collapse}
th{padding:9px 12px;font-size:10.5px;text-transform:uppercase;letter-spacing:.6px;color:var(--mu);font-weight:700;text-align:left;border-bottom:1px solid var(--br)}
td{padding:10px 12px;border-bottom:1px solid rgba(255,255,255,.03);vertical-align:middle;font-size:13px}
tr:last-child td{border-bottom:none}tr:hover td{background:rgba(255,255,255,.012)}
.tbl-wrap{overflow-x:auto;-webkit-overflow-scrolling:touch}
/* labels */
label{display:block;font-size:11.5px;font-weight:700;color:var(--mu2);margin-bottom:5px;letter-spacing:.3px;text-transform:uppercase}
/* sidebar nav */
.nv{display:flex;align-items:center;gap:10px;padding:10px 12px;border-radius:var(--r8);cursor:pointer;color:var(--mu2);font-weight:600;font-size:13.5px;border:none;background:transparent;width:100%;transition:.16s;-webkit-tap-highlight-color:transparent}
.nv:hover{background:var(--c2);color:var(--tx)}.nv.on{background:var(--goldT);color:var(--gold)}
.nv .ic{font-size:15px;width:18px;text-align:center;flex-shrink:0}
/* toast */
.toast{position:fixed;top:16px;left:50%;transform:translateX(-50%);z-index:9999;padding:11px 18px;border-radius:var(--r12);font-weight:600;font-size:13.5px;display:flex;align-items:center;gap:9px;box-shadow:0 8px 32px rgba(0,0,0,.4);width:calc(100% - 32px);max-width:340px;animation:slideIn .28s ease;text-align:left}
.tg{background:#064e3b;border:1px solid #059669;color:#6ee7b7}
.tr{background:#881337;border:1px solid #e11d48;color:#fda4af}
.ta{background:#78350f;border:1px solid #d97706;color:#fcd34d}
.tb{background:#1e3a5f;border:1px solid #3b82f6;color:#93c5fd}
/* stat */
.stat{background:var(--c2);border:1px solid var(--br);border-radius:var(--r12);padding:14px 16px}
/* misc */
.sec{font-family:'Syne',sans-serif;font-size:16px;font-weight:800;color:var(--tx);display:block}
hr{border:none;height:1px;background:var(--br);margin:14px 0}
/* overlay / modal */
.overlay{position:fixed;inset:0;background:rgba(0,0,0,.75);backdrop-filter:blur(6px);display:flex;align-items:flex-end;justify-content:center;z-index:800;padding:0;animation:fadeIn .2s ease}
.modal{background:var(--c1);border:1px solid var(--br);border-radius:20px 20px 0 0;padding:24px 20px 32px;width:100%;max-width:480px;max-height:92vh;overflow-y:auto;animation:slideUp .28s cubic-bezier(.34,1.2,.64,1)}
@keyframes slideUp{from{transform:translateY(40px);opacity:0}to{transform:translateY(0);opacity:1}}
/* ── SIDEBAR (desktop drawer, mobile overlay drawer) ── */
.sidebar{width:210px;background:var(--c1);border-right:1px solid var(--br);display:flex;flex-direction:column;position:fixed;top:0;left:0;bottom:0;z-index:250;overflow-y:auto;transition:transform .25s ease}
/* ── MAIN LAYOUT ── */
.app-main{min-height:100vh;display:flex;flex-direction:column;transition:margin-left .25s ease}
.top-bar{background:var(--c1);border-bottom:1px solid var(--br);padding:12px 16px;display:flex;align-items:center;gap:12px;position:sticky;top:0;z-index:100;min-height:52px}
.page-content{flex:1;padding:16px;width:100%;max-width:1100px;margin:0 auto}
/* ── BOTTOM NAV (mobile only) ── */
.bottom-nav{display:none;position:fixed;bottom:0;left:0;right:0;z-index:200;background:var(--c1);border-top:1px solid var(--br);height:var(--nav-h);padding:0 4px;align-items:stretch}
.bn-item{flex:1;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:3px;border:none;background:transparent;color:var(--mu2);cursor:pointer;padding:6px 2px;transition:.16s;font-size:10px;font-weight:600;font-family:inherit;-webkit-tap-highlight-color:transparent;border-radius:0}
.bn-item.on{color:var(--gold)}
.bn-item .bi{font-size:21px;line-height:1}
.bn-item.on .bi{filter:drop-shadow(0 0 4px rgba(245,158,11,.5))}
/* ── FLOATING BUTTON ── */
@keyframes fabIn{0%{opacity:0;transform:translateY(20px) scale(.85)}100%{opacity:1;transform:translateY(0) scale(1)}}
@keyframes ring{0%{transform:scale(1);opacity:.7}70%{transform:scale(1.55);opacity:0}100%{opacity:0}}
@keyframes fabPulse{0%,100%{box-shadow:0 6px 28px rgba(0,136,204,.55)}50%{box-shadow:0 8px 36px rgba(0,136,204,.85),0 0 0 6px rgba(0,136,204,.15)}}
.fab-wrap{position:fixed;bottom:22px;right:16px;z-index:300;animation:fabIn .5s cubic-bezier(.34,1.56,.64,1) both}
.fab-ring{position:absolute;inset:0;border-radius:50px;border:2px solid rgba(0,136,204,.5);animation:ring 2.4s ease-out infinite;pointer-events:none}
.fab-btn{position:relative;display:flex;align-items:center;gap:9px;padding:12px 18px;border-radius:50px;background:linear-gradient(135deg,#0ea5e9,#0077b6);color:#fff;border:none;cursor:pointer;font-weight:800;font-size:13.5px;letter-spacing:.2px;box-shadow:0 6px 28px rgba(0,136,204,.55),0 2px 8px rgba(0,0,0,.3);animation:fabPulse 3s ease infinite;transition:transform .2s;white-space:nowrap;-webkit-tap-highlight-color:transparent}
.fab-btn:active{transform:scale(.95)}
.fab-icon{width:26px;height:26px;background:rgba(255,255,255,.18);border-radius:50%;display:flex;align-items:center;justify-content:center;flex-shrink:0}
.fab-dot{width:8px;height:8px;border-radius:50%;background:#22c55e;border:1.5px solid rgba(255,255,255,.7);position:absolute;top:8px;right:8px;box-shadow:0 0 6px #22c55e}
/* tg banner */
.tg-banner{background:linear-gradient(135deg,rgba(0,136,204,.18),rgba(0,100,160,.1));border:1px solid rgba(14,165,233,.3);border-radius:var(--r12);padding:14px 16px;display:flex;align-items:center;gap:12px}
/* ── DESKTOP ── */
@media (min-width:769px){
  .sidebar{transform:translateX(0)}
  .sidebar.closed{transform:translateX(-210px)}
  .app-main{margin-left:210px}.app-main.full{margin-left:0}
  .page-content{padding:20px}
  .bottom-nav{display:none !important}
  .mob-only{display:none !important}
  .fab-wrap{bottom:22px;right:20px}
}
/* ── MOBILE ── */
@media (max-width:768px){
  .sidebar{transform:translateX(-100%)}
  .sidebar.open{transform:translateX(0)}
  .app-main{margin-left:0 !important}
  .bottom-nav{display:flex}
  .page-content{padding:12px 12px;padding-bottom:calc(var(--nav-h) + 76px)}
  .desk-only{display:none !important}
  .fab-wrap{bottom:calc(var(--nav-h) + 10px);right:12px}
  .fab-btn{padding:11px 15px;font-size:13px;gap:8px}
  .top-bar{padding:10px 14px}
  input,select{font-size:16px}
  .modal{border-radius:20px 20px 0 0;padding:22px 16px 28px}
  .stat{padding:12px 14px}
  .toast{top:12px;font-size:13px}
}`;

/* ─────────────────────────────────────────────────────────
   DATA
───────────────────────────────────────────────────────── */
const K={U:"ctb_u",A:"ctb_a",S:"ctb_s",TX:"ctb_tx",M:"ctb_m",B:"ctb_b"};
const db={
  g(k){try{return JSON.parse(localStorage.getItem(k))}catch{return null}},
  s(k,v){localStorage.setItem(k,JSON.stringify(v))},
};
function boot(){
  if(!db.g(K.A)) db.s(K.A,{user:"admin",pass:btoa("Admin@1234"),name:"Admin"});
  if(!db.g(K.U)) db.s(K.U,[]);
  if(!db.g(K.TX)) db.s(K.TX,[]);
  if(!db.g(K.B)) db.s(K.B,[]);
  if(!db.g(K.M)) db.s(K.M,[]);
}
const uid=()=>"UID"+Math.floor(100000+Math.random()*900000);
const nid=()=>Date.now().toString(36)+Math.random().toString(36).slice(2,5);
const money=(n)=>Number(n||0).toLocaleString("en-IN");
const dt=(d)=>new Date(d).toLocaleString("en-IN",{day:"2-digit",month:"short",hour:"2-digit",minute:"2-digit"});
const dtFull=(d)=>d?new Date(d).toLocaleString("en-IN",{day:"2-digit",month:"short",year:"numeric",hour:"2-digit",minute:"2-digit",hour12:true}):"—";

/* ─────────────────────────────────────────────────────────
   SCHEDULE HELPERS
───────────────────────────────────────────────────────── */
function schedStatus(m){
  if(!m.startTime&&!m.endTime) return null;
  const now=Date.now(),st=m.startTime?+new Date(m.startTime):null,en=m.endTime?+new Date(m.endTime):null;
  if(st&&now<st) return "upcoming";
  if(en&&now>en) return "ended";
  if(st&&now>=st) return "live";
  return null;
}
function SchedBadge({m}){
  const s=schedStatus(m); if(!s) return null;
  const C={upcoming:{c:"bb",t:"🕐 Upcoming"},live:{c:"br",t:"🔴 Live"},ended:{c:"bm",t:"⏱ Ended"}};
  return <span className={`badge ${C[s].c}`}>{C[s].t}</span>;
}
function Countdown({to,label,col}){
  const[v,setV]=useState("");
  useEffect(()=>{
    const tick=()=>{
      const d=+new Date(to)-Date.now();
      if(d<=0){setV("Now");return;}
      const h=Math.floor(d/3600000),m=Math.floor((d%3600000)/60000),s=Math.floor((d%60000)/1000);
      setV(`${h?h+"h ":""}${m?m+"m ":""}${s}s`);
    };
    tick();const t=setInterval(tick,1000);return()=>clearInterval(t);
  },[to]);
  return <span style={{fontSize:"11.5px",color:col||"var(--gold)",fontWeight:700,fontFamily:"monospace"}}>{label}: {v}</span>;
}

/* ─────────────────────────────────────────────────────────
   ROOT
───────────────────────────────────────────────────────── */
function App(){
  const[screen,setScreen]=useState("login");
  const[sess,setSess]=useState(null);
  const[toast,setToast]=useState(null);

  useEffect(()=>{
    // Ensure proper mobile viewport
    let vm=document.querySelector('meta[name="viewport"]');
    if(!vm){vm=document.createElement('meta');vm.name='viewport';document.head.appendChild(vm);}
    vm.content='width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no';
    boot();
    const s=db.g(K.S);
    if(s){setSess(s);setScreen(s.type==="admin"?"admin":"user");}
  },[]);

  const notify=useCallback((msg,type="g")=>{
    setToast({msg,type,k:Date.now()});
    setTimeout(()=>setToast(null),3000);
  },[]);

  const login=(s)=>{setSess(s);db.s(K.S,s);setScreen(s.type==="admin"?"admin":"user");};
  const logout=()=>{setSess(null);db.s(K.S,null);setScreen("login");notify("Logged out","b");};

  return(
    <>
      <style>{CSS}</style>
      {toast&&<div className={`toast t${toast.type}`} key={toast.k}>{toast.msg}</div>}
      {screen==="login"      &&<LoginPage  onLogin={login} toReg={()=>setScreen("reg")} toAdmin={()=>setScreen("al")} notify={notify}/>}
      {screen==="reg"        &&<RegPage    onLogin={login} back={()=>setScreen("login")} notify={notify}/>}
      {screen==="al"         &&<AdminLoginPage onLogin={login} back={()=>setScreen("login")} notify={notify}/>}
      {screen==="user"  &&sess&&<UserShell  sess={sess} logout={logout} notify={notify}/>}
      {screen==="admin" &&sess&&<AdminShell sess={sess} logout={logout} notify={notify}/>}
    </>
  );
}

/* ─────────────────────────────────────────────────────────
   AUTH
───────────────────────────────────────────────────────── */
function AuthCard({icon,title,sub,children}){
  return(
    <div style={{minHeight:"100vh",display:"flex",alignItems:"center",justifyContent:"center",padding:"16px",
      background:"radial-gradient(ellipse 90% 50% at 50% -5%,rgba(245,158,11,.07),transparent)"}}>
      <div style={{width:"100%",maxWidth:"420px"}} className="fu">
        <div style={{textAlign:"center",marginBottom:"24px"}}>
          <div style={{fontSize:"34px",marginBottom:"8px"}}>{icon}</div>
          <h1 style={{fontFamily:"'Syne'",fontSize:"20px",color:"var(--gold)",fontWeight:800}}>{title}</h1>
          {sub&&<p style={{color:"var(--mu)",marginTop:"3px",fontSize:"13px"}}>{sub}</p>}
        </div>
        <div className="card" style={{padding:"24px"}}>{children}</div>
      </div>
    </div>
  );
}
function F({label,mb=14,children}){return <div style={{marginBottom:mb+"px"}}><label>{label}</label>{children}</div>;}
function PwF({value,onChange,ph="Password",onEnter}){
  const[show,setShow]=useState(false);
  return(
    <div style={{position:"relative"}}>
      <input type={show?"text":"password"} value={value} onChange={onChange} placeholder={ph} style={{paddingRight:"42px"}} onKeyDown={e=>e.key==="Enter"&&onEnter?.()}/>
      <button onClick={()=>setShow(s=>!s)} style={{position:"absolute",right:"11px",top:"50%",transform:"translateY(-50%)",background:"none",color:"var(--mu)",fontSize:"13px",padding:"3px",lineHeight:1}}>{show?"🙈":"👁️"}</button>
    </div>
  );
}

function LoginPage({onLogin,toReg,toAdmin,notify}){
  const[f,setF]=useState({u:"",p:""});
  const[busy,set]=useState(false);
  const go=()=>{
    if(!f.u||!f.p)return notify("Fill all fields","a");
    set(true);
    setTimeout(()=>{
      const us=db.g(K.U)||[];
      const u=us.find(x=>x.username.toLowerCase()===f.u.toLowerCase());
      if(!u){notify("User not found","r");set(false);return;}
      if(u.pass!==btoa(f.p)){notify("Wrong password","r");set(false);return;}
      if(u.banned){notify("Account suspended. Contact admin.","r");set(false);return;}
      onLogin({type:"user",data:u});notify(`Welcome, ${u.nickname||u.username}! 🏏`);set(false);
    },420);
  };
  return(
    <AuthCard icon="🏏" title="Cricket Toss Book" sub="Sign in to your account">
      <F label="Username"><input value={f.u} onChange={e=>setF(p=>({...p,u:e.target.value}))} placeholder="Enter username" onKeyDown={e=>e.key==="Enter"&&go()}/></F>
      <F label="Password" mb={20}><PwF value={f.p} onChange={e=>setF(p=>({...p,p:e.target.value}))} onEnter={go}/></F>
      <button className="btn btn-gold btn-w btn-lg" onClick={go} disabled={busy}>{busy?"Signing in…":"Sign In"}</button>
      <hr/>
      <div style={{display:"flex",flexDirection:"column",gap:"8px"}}>
        <button className="btn btn-ghost btn-w" onClick={toReg}>📝 Create Account</button>
        <button className="btn btn-ghost btn-w btn-sm" onClick={toAdmin} style={{color:"var(--mu)",fontSize:"12px"}}>🛡️ Admin Panel</button>
      </div>
    </AuthCard>
  );
}

function RegPage({onLogin,back,notify}){
  const[f,setF]=useState({u:"",nick:"",p:"",c:""});
  const[busy,set]=useState(false);
  const str=useMemo(()=>f.p?[f.p.length>=6,/[A-Z
