<html lang="pt-br">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Calculadora de Compras</title>
<style>
    body {
        font-family: Arial, sans-serif;
        margin: 0;
        padding: 15px;
        background: #f5f5f5;
        display: flex;
        flex-direction: column;
        height: 100vh;
        font-size: 14px;
    }
    h2 { text-align: center; font-size: 18px; }
    input, button {
        padding: 6px;
        margin: 3px;
        font-size: 14px;
    }
    table {
        width: 100%;
        border-collapse: collapse;
        margin-top: 10px;
        table-layout: fixed;
        font-size: 12px;
    }
    th, td {
        border: 1px solid #ccc;
        padding: 5px;
        text-align: center;
        word-wrap: break-word;
    }
    .quantidade-controle {
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 3px;
    }
    .quantidade-controle button {
        padding: 3px;
        width: 25px;
        height: 25px;
        font-size: 16px;
    }
    .icone {
        background: none;
        border: none;
        cursor: pointer;
        font-size: 14px;
        padding: 2px;
    }
    .rodape {
        margin-top: auto;
        padding: 10px;
        background: #ddd;
        text-align: center;
        font-size: 14px;
    }
    .popup {
        position: fixed;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        background: white;
        padding: 15px;
        border: 2px solid red;
        text-align: center;
        display: none;
        z-index: 999;
        font-size: 14px;
    }
    .export-btn {
        position: fixed;
        top: 10px;
        right: 10px;
        background: #1976d2;
        color: #fff;
        border: none;
        padding: 4px 6px;
        border-radius: 5px;
        font-size: 11px;
        cursor: pointer;
        z-index: 1000;
    }
    .reset-btn {
        position: fixed;
        top: 10px;
        left: 10px;
        background: #d32f2f;
        color: #fff;
        border: none;
        padding: 4px 6px;
        border-radius: 5px;
        font-size: 11px;
        cursor: pointer;
        z-index: 1000;
    }
</style>
</head>
<body>

<button class="reset-btn" onclick="reiniciarPagina()">Reiniciar</button>
<button class="export-btn" onclick="exportarCupom()">Exportar</button>

<h2>Calculadora de Compras</h2>
<input type="text" id="produto" placeholder="Produto">
<input type="text" id="marca" placeholder="Marca">
<input type="text" id="preco" placeholder="Preço" oninput="mascaraMoeda(this)">
<div class="quantidade-controle">
    <button onclick="alterarQuantidade(-1)">-</button>
    <input type="number" id="quantidade" value="1" min="1" style="width: 40px; text-align:center;">
    <button onclick="alterarQuantidade(1)">+</button>
</div>
<button onclick="adicionarItem()">Adicionar</button>

<table id="tabelaCompras">
    <thead>
        <tr>
            <th>Produto</th>
            <th>Marca</th>
            <th>Preço</th>
            <th>Qtd</th>
            <th>Total</th>
            <th>Ações</th>
        </tr>
    </thead>
    <tbody></tbody>
    <tfoot>
        <tr>
            <td colspan="4" style="text-align:right; font-weight:bold;">Total Geral:</td>
            <td colspan="2" id="totalGeral" style="text-align:center; font-weight:bold;">R$ 0,00</td>
        </tr>
    </tfoot>
</table>

<div class="rodape">
    Saldo: R$ <input type="text" id="saldo" value="0" style="width:80px;" oninput="mascaraMoeda(this)"> |
    Restante: <span id="restante">0,00</span>
</div>
<div id="popup" class="popup" onclick="this.style.display='none'">Saldo ultrapassado!</div>

<script>
let totalGeral = 0;

// Máscara monetária dinâmica (preço e saldo)
function mascaraMoeda(campo) {
    let valor = campo.value.replace(/\D/g,''); // remove tudo que não for número
    if(valor === '') valor = '0';
    let numero = parseFloat(valor)/100;
    campo.value = numero.toFixed(2).replace('.', ',');
}

// Converte campo monetário para número
function parseValor(campo) {
    return parseFloat(campo.value.replace('.', '').replace(',', '.')) || 0;
}

function alterarQuantidade(valor) {
    let qtd = document.getElementById('quantidade');
    let novaQtd = parseInt(qtd.value) + valor;
    if (novaQtd >= 1) qtd.value = novaQtd;
}

function adicionarItem() {
    let produto = document.getElementById('produto').value;
    let marca = document.getElementById('marca').value;
    let preco = parseValor(document.getElementById('preco'));
    let quantidade = parseInt(document.getElementById('quantidade').value);

    if (!produto || preco <= 0 || quantidade < 1) {
        alert("Preencha os campos corretamente!");
        return;
    }

    let total = preco * quantidade;
    totalGeral += total;
    atualizarTotalGeral();
    atualizarRestante();

    let tabela = document.getElementById('tabelaCompras').querySelector('tbody');
    let linha = tabela.insertRow();
    linha.innerHTML = `
        <td>${produto}</td>
        <td>${marca}</td>
        <td>R$ ${preco.toFixed(2).replace('.', ',')}</td>
        <td>${quantidade}</td>
        <td>R$ ${(total).toFixed(2).replace('.', ',')}</td>
        <td>
            <button class="icone" onclick="editarItem(this)">✏️</button>
            <button class="icone" onclick="removerItem(this, ${total})">🗑️</button>
        </td>
    `;
    verificarSaldo();
    limparCampos();
}

function removerItem(botao, valor) {
    totalGeral -= valor;
    atualizarTotalGeral();
    atualizarRestante();
    botao.parentNode.parentNode.remove();
}

function editarItem(botao) {
    let linha = botao.parentNode.parentNode;
    document.getElementById('produto').value = linha.cells[0].innerText;
    document.getElementById('marca').value = linha.cells[1].innerText;
    document.getElementById('preco').value = linha.cells[2].innerText.replace('R$ ', '');
    document.getElementById('quantidade').value = parseInt(linha.cells[3].innerText);
    removerItem(botao, parseFloat(linha.cells[4].innerText.replace('R$ ', '').replace(',', '.')));
}

function atualizarRestante() {
    let saldo = parseValor(document.getElementById('saldo'));
    let restante = saldo - totalGeral;
    document.getElementById('restante').innerText = restante.toFixed(2).replace('.', ',');
}

function atualizarTotalGeral() {
    document.getElementById('totalGeral').innerText = `R$ ${totalGeral.toFixed(2).replace('.', ',')}`;
}

function verificarSaldo() {
    let saldo = parseValor(document.getElementById('saldo'));
    if (totalGeral > saldo) {
        document.getElementById('popup').style.display = 'block';
    }
}

function limparCampos() {
    document.getElementById('produto').value = "";
    document.getElementById('marca').value = "";
    document.getElementById('preco').value = "";
    document.getElementById('quantidade').value = 1;
}

function reiniciarPagina() {
    totalGeral = 0;
    document.querySelector('#tabelaCompras tbody').innerHTML = '';
    atualizarTotalGeral();
    document.getElementById('saldo').value = '0,00';
    document.getElementById('restante').innerText = '0,00';
    limparCampos();
}

function exportarCupom() {
    const rows = document.querySelectorAll('#tabelaCompras tbody tr');
    let texto = "**** CUPOM DE COMPRA ****\n";
    texto += "Data: " + new Date().toLocaleString('pt-BR') + "\n\n";
    let total = 0;

    rows.forEach(row => {
        const produto = row.cells[0].innerText || '';
        const marca = row.cells[1].innerText || '';
        const qtdText = row.cells[3].innerText || '0';
        const subtotalText = (row.cells[4].innerText || 'R$ 0,00').replace('R$ ', '').replace(',', '.');
        const qtd = parseInt(qtdText) || 0;
        const subtotal = parseFloat(subtotalText) || 0;
        texto += `${produto} (${marca}) x${qtd} - R$ ${subtotal.toFixed(2).replace('.', ',')}\n`;
        total += subtotal;
    });

    const saldoInput = parseValor(document.getElementById('saldo'));
    const restante = saldoInput - total;

    texto += `\nTOTAL: R$ ${total.toFixed(2).replace('.', ',')}\nSALDO: R$ ${saldoInput.toFixed(2).replace('.', ',')}\nRESTANTE: R$ ${restante.toFixed(2).replace('.', ',')}\n`;
    texto += "\n***************************\n";

    const blob = new Blob([texto], { type: "text/plain;charset=utf-8" });
    const link = document.createElement("a");
    link.href = URL.createObjectURL(blob);
    link.download = "cupom.txt";
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
}
</script>
</body>
</html>
