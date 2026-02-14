<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>NostrKut - Social Descentralizada</title>
    <script src="https://unpkg.com/nostr-tools/lib/nostr.bundle.js"></script>
    <style>
        :root { --blue-main: #6D84B4; --blue-light: #BFD0EA; --bg-gray: #E9EFF4; --pink: #E91E63; }
        body { background: var(--bg-gray); font-family: Arial, sans-serif; margin: 0; font-size: 12px; }
        
        /* Header */
        header { background: var(--blue-light); padding: 10px 40px; border-bottom: 2px solid var(--blue-main); display: flex; justify-content: space-between; align-items: center; }
        .logo { color: var(--pink); font-weight: bold; font-size: 26px; text-decoration: none; font-style: italic; }
        .nav-top a { color: var(--blue-main); text-decoration: none; margin: 0 10px; font-weight: bold; }

        /* Layout */
        .wrapper { display: flex; max-width: 1000px; margin: 20px auto; gap: 15px; }
        .col-left { width: 200px; }
        .col-main { flex: 1; background: white; border: 1px solid #B4C5E4; min-height: 600px; }
        .col-right { width: 230px; }

        /* Blocos */
        .box { background: white; border: 1px solid #B4C5E4; padding: 10px; margin-bottom: 15px; }
        .header-box { background: var(--blue-light); padding: 5px; font-weight: bold; color: #444; margin-bottom: 10px; display: flex; justify-content: space-between; }
        .profile-pic { width: 100%; border: 1px solid #CCC; display: block; }
        
        /* Menu Lateral */
        .side-menu a { display: block; padding: 4px; color: var(--blue-main); text-decoration: none; border-bottom: 1px solid #EEE; }
        .side-menu a:hover { background: #F0F0F0; }

        /* Scraps e Itens */
        .item-list { padding: 10px; border-bottom: 1px dashed #CCC; }
        .item-list:nth-child(even) { background: #F9F9F9; }
        .btn-pink { background: var(--pink); color: white; border: none; padding: 5px 12px; cursor: pointer; font-weight: bold; }
        
        /* Grid Amigos */
        .grid-9 { display: grid; grid-template-columns: repeat(3, 1fr); gap: 5px; text-align: center; }
        .grid-9 img { width: 60px; height: 60px; object-fit: cover; border: 1px solid #CCC; }
        .grid-9 span { display: block; font-size: 10px; color: var(--blue-main); overflow: hidden; text-overflow: ellipsis; }

        /* Abas */
        .tab-content { display: none; padding: 15px; }
        .tab-content.active { display: block; }
        .tab-nav { background: #DDE5F1; display: flex; border-bottom: 1px solid var(--blue-main); }
        .tab-nav div { padding: 8px 15px; cursor: pointer; color: var(--blue-main); font-weight: bold; }
        .tab-nav div.active { background: white; border: 1px solid var(--blue-main); border-bottom: none; position: relative; top: 1px; }
    </style>
</head>
<body>

<header>
    <a href="javascript:void(0)" onclick="switchTab('perfil')" class="logo">nostrkut</a>
    <div class="nav-top">
        <span id="welcome-msg">Página de login:</span>
        <button id="btn-auth" class="btn-pink" onclick="toggleAuth()">Login Nostr</button>
    </div>
</header>

<div class="wrapper">
    <div class="col-left">
        <div class="box">
            <img id="side-pic" src="https://via.placeholder.com/180x180?text=Foto" class="profile-pic">
            <div class="side-menu">
                <a href="javascript:void(0)" onclick="switchTab('perfil')">Perfil</a>
                <a href="javascript:void(0)" onclick="switchTab('scraps')">Recados</a>
                <a href="javascript:void(0)" onclick="switchTab('comunidades')">Comunidades</a>
            </div>
        </div>
    </div>

    <div class="col-main">
        <div class="tab-nav">
            <div id="tab-link-perfil" class="active" onclick="switchTab('perfil')">Início</div>
            <div id="tab-link-scraps" onclick="switchTab('scraps')">Recados</div>
            <div id="tab-link-comunidades" onclick="switchTab('comunidades')">Comunidades</div>
        </div>

        <div id="tab-perfil" class="tab-content active">
            <h2 id="main-name" style="color: var(--blue-main);">Carregando...</h2>
            <div style="background:#F3F3F3; padding:10px; border-left: 4px solid var(--pink);">
                <b>Status:</b> <span id="main-status">Diga algo ao mundo via Nostr Kind 0.</span>
            </div>
            <div class="header-box" style="margin-top:20px;">Social</div>
            <p><b>Confiavel:</b> 🧊🧊🧊 <b>Legal:</b> 🧊🧊🧊 <b>Sexy:</b> 🧊🧊🧊</p>
            <div class="header-box">Sobre mim</div>
            <div id="main-bio" style="line-height:1.6;">...</div>
        </div>

        <div id="tab-scraps" class="tab-content">
            <div class="header-box">meu mural de recados</div>
            <div id="list-scraps">Buscando recados na rede...</div>
            <hr>
            <textarea id="input-scrap" style="width:100%; height:60px;" placeholder="Escreva um scrap para este perfil..."></textarea>
            <button class="btn-pink" onclick="postScrap()">enviar</button>
        </div>

        <div id="tab-comunidades" class="tab-content">
            <div class="header-box">minhas comunidades</div>
            <div class="item-list"><b>Eu odeio acordar cedo</b> (1.2M membros)</div>
            <div class="item-list"><b>Sou legal, não estou te dando mole</b> (800k membros)</div>
            <div class="item-list"><b>Nostr Brasil</b> (Recém criada)</div>
        </div>
    </div>

    <div class="col-right">
        <div class="box">
            <div class="header-box">amigos <a href="#" style="font-size:10px; font-weight:normal;">(ver todos)</a></div>
            <div class="grid-9" id="grid-friends">
                </div>
        </div>
    </div>
</div>

<script>
    const RELAY_ADDR = "wss://relay.damus.io";
    let myPubkey = "32e1827635450ebb3c5a7d12c1f8e7b2b514439ac10a67eef3d9fd9c5c68e245"; // Default

    async function toggleAuth() {
        if (!window.nostr) return alert("Instale Alby ou Nos2x!");
        myPubkey = await window.nostr.getPublicKey();
        document.getElementById('btn-auth').innerText = "Logado";
        loadAll();
    }

    function switchTab(id) {
        document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
        document.querySelectorAll('.tab-nav div').forEach(t => t.classList.remove('active'));
        document.getElementById('tab-' + id).classList.add('active');
        document.getElementById('tab-link-' + id).classList.add('active');
    }

    async function loadAll() {
        const relay = window.NostrTools.relayInit(RELAY_ADDR);
        await relay.connect();

        // 1. Carregar Perfil (Kind 0)
        relay.sub([{ kinds: [0], authors: [myPubkey] }]).on('event', ev => {
            const p = JSON.parse(ev.content);
            document.getElementById('side-pic').src = p.picture || "https://via.placeholder.com/180";
            document.getElementById('main-name').innerText = p.display_name || p.name || "Usuário";
            document.getElementById('main-status').innerText = p.about?.substring(0, 50) || "...";
            document.getElementById('main-bio').innerText = p.about || "Sem bio disponível.";
            document.getElementById('welcome-msg').innerText = "Olá, " + (p.name || "Nostrkutiano");
        });

        // 2. Carregar Amigos (Kind 3)
        const grid = document.getElementById('grid-friends');
        relay.sub([{ kinds: [3], authors: [myPubkey], limit: 1 }]).on('event', ev => {
            grid.innerHTML = '';
            ev.tags.slice(0, 9).forEach(t => {
                if(t[0] === 'p') {
                    grid.innerHTML += `<div class="friend-item"><img src="https://robohash.org/${t[1]}?set=set4"><span>${t[1].substring(0,5)}</span></div>`;
                }
            });
        });

        // 3. Carregar Scraps (Kind 1)
        const sList = document.getElementById('list-scraps');
        relay.sub([{ kinds: [1], "#p": [myPubkey], limit: 10 }]).on('event', ev => {
            if(sList.innerText.includes('Buscando')) sList.innerHTML = '';
            const d = document.createElement('div');
            d.className = 'item-list';
            d.innerHTML = `<b>${ev.pubkey.substring(0,8)}:</b> ${ev.content} <br><small style="color:#999">${new Date(ev.created_at*1000).toLocaleDateString()}</small>`;
            sList.prepend(d);
        });
    }

    async function postScrap() {
        if(!window.nostr) return alert("Faça login!");
        const txt = document.getElementById('input-scrap').value;
        const event = {
            kind: 1,
            created_at: Math.floor(Date.now()/1000),
            tags: [['p', myPubkey]],
            content: txt
        };
        const signed = await window.nostr.signEvent(event);
        const relay = window.NostrTools.relayInit(RELAY_ADDR);
        await relay.connect();
        await relay.publish(signed);
        alert("Scrap enviado!");
        document.getElementById('input-scrap').value = "";
    }

    // Iniciar com perfil default
    loadAll();
</script>
</body>
</html>
