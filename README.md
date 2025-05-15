# jogo-da-vida
projeto feito por alunos de Analise e desenvolvimento de sistemas da UDF
[Life game (2).zip](https://github.com/user-attachments/files/20233378/Life.game.2.zip)
// --- codigo JS ---- //
// --- Configuração do tabuleiro e cartas já estão no index.html ---

// --- Configuração dinâmica dos jogadores ---

const playerColors = ["#1976d2", "#d32f2f", "#388e3c", "#fbc02d"];
let players = [];
let currentPlayer = 0;
let diceRolled = false;

// Função para criar campos de configuração de jogadores
function renderPlayerSetupFields() {
    const container = document.getElementById('players-setup');
    container.innerHTML = '';
    for (let i = 0; i < window.setupPlayerCount; i++) {
        container.innerHTML += `
            <div style="margin-bottom:10px;">
                <label>Nome do Jogador ${i+1}: <input type="text" required value="Jogador ${i+1}" id="player-name-${i}" style="width:120px"></label>
                <label>Cor: 
                    <select id="player-color-${i}">
                        ${playerColors.map((color, idx) => `<option value="${color}" ${idx===i?'selected':''} style="color:${color};background:${color};">${color}</option>`).join('')}
                    </select>
                </label>
            </div>
        `;
    }
}

// Inicialização do setup
window.setupPlayerCount = 2;
renderPlayerSetupFields();

document.getElementById('add-player').onclick = () => {
    if (window.setupPlayerCount < 4) {
        window.setupPlayerCount++;
        renderPlayerSetupFields();
    }
};
document.getElementById('remove-player').onclick = () => {
    if (window.setupPlayerCount > 2) {
        window.setupPlayerCount--;
        renderPlayerSetupFields();
    }
};

document.getElementById('setup-form').onsubmit = function(e) {
    e.preventDefault();
    players = [];
    let usedColors = [];
    let usedNames = [];
    for (let i = 0; i < window.setupPlayerCount; i++) {
        const name = document.getElementById(`player-name-${i}`).value.trim() || `Jogador ${i+1}`;
        const color = document.getElementById(`player-color-${i}`).value;
        if (usedColors.includes(color)) {
            alert("Cada jogador deve ter uma cor diferente!");
            return;
        }
        if (usedNames.includes(name.toLowerCase())) {
            alert("Cada jogador deve ter um nome diferente!");
            return;
        }
        usedColors.push(color);
        usedNames.push(name.toLowerCase());
        players.push({
            name,
            color,
            position: 0,
            money: 1500,
            properties: [],
            inJail: false,
            jailTurns: 0
        });
    }
    document.getElementById('setup').style.display = 'none';
    document.getElementById('controls').style.display = '';
    document.getElementById('players').style.display = '';
    document.getElementById('game-board').style.display = '';
    document.getElementById('log').style.display = '';
    document.getElementById('dice').style.display = 'flex'; // <-- Mostra o dado
    startGame();
};

// Esconde elementos do jogo até o setup terminar
document.getElementById('controls').style.display = 'none';
document.getElementById('players').style.display = 'none';
document.getElementById('game-board').style.display = 'none';
document.getElementById('log').style.display = 'none';
document.getElementById('dice').style.display = 'none'; // <-- Esconde o dado

// Chame isso após o setup para iniciar o jogo normalmente
function startGame() {
    currentPlayer = 0;
    diceRolled = false;
    renderBoard();
    renderPlayers();
    log(`<b>Vez de ${players[currentPlayer].name}</b>`);
}

// Tabuleiro simplificado (40 casas, igual Monopoly)
const board = [
    { name: "Início", type: "corner" },
    { name: "Mediterranean Ave", type: "property", price: 60, rent: 2 },
    { name: "Chance", type: "chance" },
    { name: "Baltic Ave", type: "property", price: 60, rent: 4 },
    { name: "Imposto", type: "tax", amount: 200 },
    { name: "Reading Railroad", type: "property", price: 200, rent: 25 },
    { name: "Oriental Ave", type: "property", price: 100, rent: 6 },
    { name: "Chance", type: "chance" },
    { name: "Vermont Ave", type: "property", price: 100, rent: 6 },
    { name: "Connecticut Ave", type: "property", price: 120, rent: 8 },
    { name: "Prisão", type: "corner" },
    { name: "St. Charles Place", type: "property", price: 140, rent: 10 },
    { name: "Electric Company", type: "property", price: 150, rent: 12 },
    { name: "States Ave", type: "property", price: 140, rent: 10 },
    { name: "Virginia Ave", type: "property", price: 160, rent: 12 },
    { name: "Pennsylvania Railroad", type: "property", price: 200, rent: 25 },
    { name: "St. James Place", type: "property", price: 180, rent: 14 },
    { name: "Chance", type: "chance" },
    { name: "Tennessee Ave", type: "property", price: 180, rent: 14 },
    { name: "New York Ave", type: "property", price: 200, rent: 16 },
    { name: "Parada Livre", type: "corner" },
    { name: "Kentucky Ave", type: "property", price: 220, rent: 18 },
    { name: "Chance", type: "chance" },
    { name: "Indiana Ave", type: "property", price: 220, rent: 18 },
    { name: "Illinois Ave", type: "property", price: 240, rent: 20 },
    { name: "B&O Railroad", type: "property", price: 200, rent: 25 },
    { name: "Atlantic Ave", type: "property", price: 260, rent: 22 },
    { name: "Ventnor Ave", type: "property", price: 260, rent: 22 },
    { name: "Water Works", type: "property", price: 150, rent: 12 },
    { name: "Marvin Gardens", type: "property", price: 280, rent: 24 },
    { name: "Vá para Prisão", type: "corner" },
    { name: "Pacific Ave", type: "property", price: 300, rent: 26 },
    { name: "North Carolina Ave", type: "property", price: 300, rent: 26 },
    { name: "Chance", type: "chance" },
    { name: "Pennsylvania Ave", type: "property", price: 320, rent: 28 },
    { name: "Short Line", type: "property", price: 200, rent: 25 },
    { name: "Chance", type: "chance" },
    { name: "Park Place", type: "property", price: 350, rent: 35 },
    { name: "Imposto de Luxo", type: "tax", amount: 100 },
    { name: "Boardwalk", type: "property", price: 400, rent: 50 }
];

// Cartas de chance simples
const chanceCards = [
    { text: "Avance até o Início e receba R$200", action: (p) => { p.position = 0; p.money += 200; } },
    { text: "Pague R$50 de taxas", action: (p) => { p.money -= 50; } },
    { text: "Receba R$100 de herança", action: (p) => { p.money += 100; } },
    { text: "Vá para a Prisão", action: (p) => { p.position = 10; p.inJail = true; p.jailTurns = 0; } }
];

// O JS abaixo assume que as variáveis board, chanceCards, players, playerColors, currentPlayer, diceRolled já existem no escopo global (index.html).

// Se preferir, remova as definições duplicadas do index.html e mantenha apenas aqui.

function renderBoard() {
    const boardDiv = document.getElementById('game-board');
    boardDiv.innerHTML = '';
    for (let y = 0; y < 11; y++) {
        for (let x = 0; x < 11; x++) {
            let idx = -1;
            if (y === 0) idx = x;
            else if (x === 10) idx = 10 + y;
            else if (y === 10) idx = 30 - x;
            else if (x === 0) idx = 30 + (10 - y);
            let cell = document.createElement('div');
            cell.className = 'cell';
            if (idx >= 0 && idx < board.length) {
                let space = board[idx];
                cell.innerHTML = `<div>${space.name}</div>`;
                cell.dataset.idx = idx;
                cell.setAttribute('data-name', space.name); // Para CSS de emoji
                if (space.type === "corner") cell.classList.add('corner');
                if (space.type === "property") {
                    cell.classList.add('property');
                    if (space.owner !== undefined) cell.classList.add('owned');
                }
                if (space.type === "chance") cell.classList.add('chance');
                if (space.type === "tax") cell.classList.add('tax');
            }
            boardDiv.appendChild(cell);
        }
    }
    // Renderiza jogadores
    players.forEach((p, i) => {
        let idx = p.position;
        let cell = boardDiv.querySelector(`.cell[data-idx="${idx}"]`);
        if (cell) {
            let token = document.createElement('div');
            token.className = 'player';
            token.style.background = p.color;
            token.title = p.name;
            token.style.left = `${2 + i*20}px`;
            token.style.bottom = `${2 + i*20}px`;
            cell.appendChild(token);
        }
    });
}

function renderPlayers() {
    const playersDiv = document.getElementById('players');
    playersDiv.innerHTML = players.map((p, i) =>
        `<div class="${i === currentPlayer ? 'current-player' : ''}" style="background:${p.color};">
            <b>${p.name}</b><br>
            Dinheiro: R$${p.money}<br>
            Propriedades: ${p.properties.length}
        </div>`
    ).join('');
}

function log(msg) {
    const logDiv = document.getElementById('log');
    logDiv.innerHTML += msg + "<br>";
    logDiv.scrollTop = logDiv.scrollHeight;
}

function showModal(msg, cb) {
    const modal = document.getElementById('modal');
    const content = document.getElementById('modal-content');
    content.innerHTML = msg + '<br><br><button class="btn" id="close-modal">OK</button>';
    modal.style.display = 'flex';
    document.getElementById('close-modal').onclick = () => {
        modal.style.display = 'none';
        if (cb) cb();
    };
}

function rollDice() {
    if (diceRolled) return;
    let player = players[currentPlayer];
    const diceDiv = document.getElementById('dice');

    // Lógica de prisão: se o jogador está preso, conta os turnos
    if (player.inJail) {
        player.jailTurns++;
        log(`${player.name} está na prisão (turno ${player.jailTurns}/3).`);
        if (player.jailTurns >= 3) {
            player.inJail = false;
            player.jailTurns = 0;
            log(`${player.name} saiu da prisão após 3 turnos.`);
        } else {
            diceRolled = true;
            setTimeout(() => {
                endTurn();
            }, 800);
            return;
        }
    }

    // Animação de rolagem
    let rolls = 20;
    let interval = setInterval(() => {
        let d1 = Math.ceil(Math.random() * 6);
        let d2 = Math.ceil(Math.random() * 6);
        diceDiv.textContent = `🎲 ${d1} + ${d2}`;
        rolls--;
        if (rolls === 0) {
            clearInterval(interval);
            // Agora rola de verdade
            let d1 = Math.ceil(Math.random() * 6);
            let d2 = Math.ceil(Math.random() * 6);
            diceDiv.textContent = `🎲 ${d1} + ${d2}`;
            let move = d1 + d2;
            log(`${player.name} rolou ${d1} e ${d2} (total: ${move})`);
            let oldPos = player.position;
            let newPos = (player.position + move) % board.length;
            let passedStart = newPos < player.position;
            diceRolled = true;
            animatePawnMove(currentPlayer, player.position, newPos, () => {
                player.position = newPos;
                if (passedStart) {
                    player.money += 200;
                    log(`${player.name} passou pelo Início e recebeu R$200`);
                }
                renderBoard();
                handleSpace();
            });
        }
    }, 70);
}

function handleSpace() {
    let player = players[currentPlayer];
    let space = board[player.position];
    document.getElementById('buy-btn').disabled = true;
    document.getElementById('sell-btn').disabled = player.properties.length === 0;
    document.getElementById('end-btn').disabled = false;
    if (space.type === "property") {
        if (space.owner === undefined) {
            document.getElementById('buy-btn').disabled = false;
            log(`${player.name} pode comprar ${space.name} por R$${space.price}`);
        } else if (space.owner !== currentPlayer) {
            let rent = space.rent;
            player.money -= rent;
            players[space.owner].money += rent;
            log(`${player.name} pagou R$${rent} de aluguel para ${players[space.owner].name}`);
        }
    } else if (space.type === "tax") {
        player.money -= space.amount;
        log(`${player.name} pagou R$${space.amount} de imposto.`);
    } else if (space.type === "chance") {
        let card = chanceCards[Math.floor(Math.random() * chanceCards.length)];
        log(`<b>Carta Sorte/Azar:</b> ${card.text}`);
        card.action(player);
        renderBoard();
    } else if (space.name === "Vá para Prisão") {
        player.position = 10;
        player.inJail = true;
        player.jailTurns = 0;
        log(`${player.name} foi para a prisão!`);
        renderBoard();
    }
    renderPlayers();
    checkBankruptcy();
}

function buyProperty() {
    let player = players[currentPlayer];
    let space = board[player.position];
    if (space.type === "property" && space.owner === undefined && player.money >= space.price) {
        player.money -= space.price;
        player.properties.push(player.position);
        space.owner = currentPlayer;
        log(`${player.name} comprou ${space.name} por R$${space.price}`);
        renderBoard();
        renderPlayers();
        document.getElementById('buy-btn').disabled = true;
        document.getElementById('sell-btn').disabled = false;
    }
}

function sellProperty() {
    let player = players[currentPlayer];
    if (player.properties.length === 0) return;
    let propIdx = prompt("Digite o número da propriedade para vender (1 a " + player.properties.length + "):");
    let idx = parseInt(propIdx) - 1;
    if (isNaN(idx) || idx < 0 || idx >= player.properties.length) return;
    let pos = player.properties[idx];
    let space = board[pos];
    player.money += Math.floor(space.price / 2);
    log(`${player.name} vendeu ${space.name} por R$${Math.floor(space.price / 2)}`);
    space.owner = undefined;
    player.properties.splice(idx, 1);
    renderBoard();
    renderPlayers();
    document.getElementById('sell-btn').disabled = player.properties.length === 0;
}

function endTurn() {
    diceRolled = false;
    document.getElementById('buy-btn').disabled = true;
    document.getElementById('sell-btn').disabled = true;
    document.getElementById('end-btn').disabled = true;
    currentPlayer = (currentPlayer + 1) % players.length;
    log(`<b>Vez de ${players[currentPlayer].name}</b>`);
    renderPlayers();
}

function checkBankruptcy() {
    let player = players[currentPlayer];
    if (player.money < 0) {
        showModal(`${player.name} faliu! Fim de jogo.`, () => location.reload());
    }
}

// Inicialização
document.getElementById('roll-btn').onclick = rollDice;
document.getElementById('buy-btn').onclick = buyProperty;
document.getElementById('sell-btn').onclick = sellProperty;
document.getElementById('end-btn').onclick = endTurn;

renderBoard();
renderPlayers();
log(`<b>Vez de ${players[currentPlayer].name}</b>`);

function animatePawnMove(playerIdx, from, to, callback) {
    const boardDiv = document.getElementById('game-board');
    let steps = [];
    let pos = from;
    while (pos !== to) {
        pos = (pos + 1) % board.length;
        steps.push(pos);
    }
    let i = 0;
    function step() {
        players[playerIdx].position = steps[i];
        renderBoard();
        if (i < steps.length - 1) {
            i++;
            setTimeout(step, 120);
        } else if (callback) {
            callback();
        }
    }
    if (steps.length > 0) step();
    else if (callback) callback();
}
