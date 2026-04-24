<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Sistema de Tempo de Serviço Militar</title>

<style>
body {
  font-family: Arial;
  max-width: 900px;
  margin: auto;
  padding: 20px;
  background: #f4f6f9;
}

h2 { text-align: center; }

.card {
  background: white;
  padding: 15px;
  margin-bottom: 15px;
  border-radius: 6px;
  border: 1px solid #ddd;
}

label { font-weight: bold; }

input, select {
  width: 100%;
  padding: 6px;
  margin-top: 4px;
  margin-bottom: 10px;
}

button {
  width: 100%;
  padding: 10px;
  background: #2c7be5;
  color: white;
  border: none;
  cursor: pointer;
}

.resultado {
  background: #eef3ff;
  padding: 15px;
  border-radius: 6px;
  white-space: pre-line;
}

.post {
  background: #fff;
  padding: 15px;
  margin-top: 10px;
  border-left: 4px solid #2c7be5;
}
</style>

</head>

<body>

<h2>Sistema de Tempo de Serviço Militar</h2>

<div class="card">
  <label>Data de Praça:</label>
  <input type="text" id="praca" placeholder="dd/mm/aaaa" oninput="mascaraData(this)">

  <label>Início da Folha:</label>
  <input type="text" id="folhaInicio" placeholder="dd/mm/aaaa" oninput="mascaraData(this)">

  <label>Fim da Folha:</label>
  <input type="text" id="folhaFim" placeholder="dd/mm/aaaa" oninput="mascaraData(this)">
</div>

<div class="card">
  <h3>Alterações</h3>

  <select id="tipo">
    <option value="curso">Curso/Missão (não arregimentado)</option>
    <option value="ltip">LTIP (não computa)</option>
    <option value="prisao">Prisão (não computa)</option>
  </select>

  <input type="text" id="altInicio" placeholder="Início (dd/mm/aaaa)" oninput="mascaraData(this)">
  <input type="text" id="altFim" placeholder="Fim (dd/mm/aaaa)" oninput="mascaraData(this)">

  <button onclick="add()">Adicionar</button>

  <div id="lista"></div>
</div>

<div class="card">
  <button onclick="calcular()">Calcular</button>
  <button onclick="copiar()">Copiar Resultado</button>
</div>

<div class="resultado" id="res"></div>

<div class="card">
  <h3>📚 Consulta – Medalha Militar (Exército Brasileiro)</h3>

  <div class="post">
    <b>Base legal:</b><br>
    Lei nº 6.880/1980 (Estatuto dos Militares) + normas do DGP
    <br><br>

    <b>📌 O que é:</b><br>
    A Medalha Militar é concedida por tempo de bons serviços prestados ao Exército, sem punições impeditivas.
    <br><br>

    <b>⏱️ Tempo necessário:</b><br>
    - 10 anos → Bronze<br>
    - 20 anos → Prata<br>
    - 30 anos → Ouro
    <br><br>

    <b>✔ Conta como tempo:</b><br>
    - Tempo de serviço ativo<br>
    - Tempo arregimentado<br>
    - Parte do tempo não arregimentado (curso, missão, etc.)
    <br><br>

    <b>❌ Não conta:</b><br>
    - LTIP (Licença Interesse Particular)<br>
    - Deserção<br>
    - Prisão não computável<br>
    - Tempo perdido disciplinarmente
    <br><br>

    <b>⚠️ Condições obrigatórias:</b><br>
    - Boa conduta<br>
    - Sem punições impeditivas no período<br>
    - Tempo contínuo válido
    <br><br>

    <b>📌 Observações importantes:</b><br>
    - O tempo para medalha é diferente do TTES<br>
    - Nem todo tempo de serviço entra para medalha<br>
    - Deve ser conferido com assentamentos individuais
  </div>
</div>

<script>
let alteracoes = [];

function mascaraData(campo) {
  let v = campo.value.replace(/\D/g, '');
  if (v.length >= 2) v = v.slice(0,2) + '/' + v.slice(2);
  if (v.length >= 5) v = v.slice(0,5) + '/' + v.slice(5,9);
  campo.value = v;
}

function parseData(data) {
  let [d,m,a] = data.split("/");
  return new Date(a, m-1, d);
}

function diasEntre(i,f) {
  return Math.floor((f-i)/(1000*60*60*24))+1;
}

function diff(inicio, fim) {
  let i = new Date(inicio);
  let f = new Date(fim);

  let a = f.getFullYear()-i.getFullYear();
  let m = f.getMonth()-i.getMonth();
  let d = f.getDate()-i.getDate();

  if (d<0) {
    m--;
    d += new Date(f.getFullYear(), f.getMonth(), 0).getDate();
  }
  if (m<0) {
    a--;
    m+=12;
  }

  return `${a}a ${m}m ${d}d`;
}

function diasData(d) {
  return new Date(1970,0,d);
}

function add() {
  let tipo = document.getElementById("tipo").value;
  let i = parseData(document.getElementById("altInicio").value);
  let f = parseData(document.getElementById("altFim").value);

  alteracoes.push({tipo,i,f});

  document.getElementById("lista").innerHTML +=
    `<div>${tipo} - ${i.toLocaleDateString()} até ${f.toLocaleDateString()}</div>`;
}

function calcular() {
  let praca = parseData(document.getElementById("praca").value);
  let fi = parseData(document.getElementById("folhaInicio").value);
  let ff = parseData(document.getElementById("folhaFim").value);

  let totalFolha = diasEntre(fi,ff);
  let totalCarreira = diasEntre(praca,ff);

  let tnc=0, naoArreg=0, tncTotal=0;

  alteracoes.forEach(a=>{
    let i = a.i<fi?fi:a.i;
    let f = a.f>ff?ff:a.f;

    if(i<=f){
      let dias = diasEntre(i,f);
      if(a.tipo==="ltip"||a.tipo==="prisao") tnc+=dias;
      else naoArreg+=dias;
    }

    let diasTotal = diasEntre(a.i,a.f);
    if(a.tipo==="ltip"||a.tipo==="prisao") tncTotal+=diasTotal;
  });

  let tcFolha = totalFolha - tnc;
  let arreg = tcFolha - naoArreg;
  let ttes = totalCarreira - tncTotal;

  let texto = `
PERÍODO DA FOLHA
TC Arregimentado: ${diff(0,diasData(arreg))}
TC Não Arregimentado: ${diff(0,diasData(naoArreg))}
TNC: ${diff(0,diasData(tnc))}

TOTAL DE SERVIÇO
TTES: ${diff(0,diasData(ttes))}
Medalha Militar: ${diff(0,diasData(ttes))}
`;

  document.getElementById("res").innerText = texto;
}

function copiar(){
  navigator.clipboard.writeText(document.getElementById("res").innerText);
  alert("Copiado!");
}
</script>

</body>
</html>
