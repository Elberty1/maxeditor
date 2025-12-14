<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>MaxEvol • Key Dashboard</title>

<style>
*{box-sizing:border-box;font-family:Arial}
body{margin:0;background:#0f0f1a;color:#fff}
.header{background:linear-gradient(90deg,#7c3aed,#9333ea);padding:20px;text-align:center;font-size:22px;font-weight:bold}
.container{display:flex}
.sidebar{width:220px;background:#14142b;min-height:100vh}
.sidebar button{width:100%;padding:15px;border:none;background:none;color:#bbb;text-align:left;cursor:pointer}
.sidebar button:hover,.sidebar button.active{background:#1f1f3a;color:#fff}
.content{flex:1;padding:20px}
.card{background:#1a1a33;padding:20px;border-radius:10px;margin-bottom:20px}
input,select,button{width:100%;padding:10px;margin-top:10px;border:none;border-radius:6px}
button{background:#7c3aed;color:#fff;cursor:pointer}
button:hover{background:#9333ea}
.key{background:#22224a;padding:15px;border-radius:8px;margin-top:10px}
.key small{color:#aaa}
.key-actions{display:flex;gap:10px;margin-top:10px}
.red{background:#dc2626}
.red:hover{background:#ef4444}
.terminal{background:#000;padding:10px;border-radius:6px;font-family:monospace}
.credit{color:#22c55e;font-weight:bold}
</style>
</head>

<body>

<div class="header">MaxEvol • Key Manager</div>

<div class="container">
<div class="sidebar">
    <button class="active" onclick="showTab('gen',this)">🔑 Gerar Key</button>
    <button onclick="showTab('list',this)">📋 Keys Ativas</button>
    <button onclick="showTab('cmd',this)">💻 Terminal</button>
</div>

<div class="content">

<!-- GERAR -->
<div id="gen" class="card">
    <h2>Gerar Key</h2>
    <p>Créditos: <span class="credit" id="creditView">0</span></p>

    <input id="name" placeholder="Nome do Player">
    <select id="days">
        <option value="1">1 dia (25 créditos)</option>
        <option value="5">5 dias (55 créditos)</option>
        <option value="10">10 dias (76 créditos)</option>
        <option value="20">20 dias (100 créditos)</option>
        <option value="30">30 dias (145 créditos)</option>
    </select>

    <button onclick="generateKey()">GERAR KEY</button>
</div>

<!-- LISTA -->
<div id="list" class="card" style="display:none">
    <h2>Keys Ativas</h2>
    <div id="keys"></div>
</div>

<!-- TERMINAL -->
<div id="cmd" class="card" style="display:none">
    <h2>Terminal</h2>
    <div class="terminal">
        <div>> Digite comandos</div>
    </div>
    <input id="command" placeholder="/credits 100">
    <button onclick="runCommand()">EXECUTAR</button>
</div>

</div>
</div>

<script>
let keys = JSON.parse(localStorage.getItem("keys") || "[]");
let credits = parseInt(localStorage.getItem("credits") || "0");

const costs = {1:25,5:55,10:76,20:100,30:145};

function showTab(id,btn){
    document.querySelectorAll(".card").forEach(c=>c.style.display="none");
    document.getElementById(id).style.display="block";
    document.querySelectorAll(".sidebar button").forEach(b=>b.classList.remove("active"));
    btn.classList.add("active");
}

function randomBlock(){
    return Math.random().toString(36).substring(2,6).toUpperCase();
}

function generateKey(){
    const name = document.getElementById("name").value;
    const days = parseInt(document.getElementById("days").value);
    const cost = costs[days];

    if(!name) return alert("Digite o nome");
    if(credits < cost) return alert("Créditos insuficientes");

    credits -= cost;

    const key = `MaxEvol-${randomBlock()}-${randomBlock()}`;
    const expire = Date.now() + days*86400000;

    keys.push({name,key,expire});
    save();
}

function render(){
    document.getElementById("creditView").innerText = credits;
    const box = document.getElementById("keys");
    box.innerHTML="";
    const now = Date.now();

    keys = keys.filter(k=>k.expire>now);

    keys.forEach((k,i)=>{
        const div = document.createElement("div");
        div.className="key";
        div.innerHTML=`
            <b>${k.key}</b><br>
            <small>Nome: ${k.name}</small><br>
            <small>Expira: ${new Date(k.expire).toLocaleString()}</small>
            <div class="key-actions">
                <button onclick="renew(${i})">Renovar</button>
                <button class="red" onclick="removeKey(${i})">Cancelar</button>
            </div>`;
        box.appendChild(div);
    });
}

function save(){
    localStorage.setItem("keys",JSON.stringify(keys));
    localStorage.setItem("credits",credits);
    render();
}

function renew(i){
    keys[i].expire += 86400000;
    save();
}

function removeKey(i){
    keys.splice(i,1);
    save();
}

function runCommand(){
    const cmd = document.getElementById("command").value.trim();
    if(cmd.startsWith("/credits")){
        const value = parseInt(cmd.split(" ")[1]);
        if(!isNaN(value)){
            credits += value;
            save();
            alert("Créditos adicionados: "+value);
        }
    }
    document.getElementById("command").value="";
}

render();
</script>

</body>
</html>
