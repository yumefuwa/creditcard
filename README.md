<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>クレジットカード特典シミュレーター</title>
  <style>
    body {
      font-family: 'Helvetica Neue', sans-serif;
      margin: 2em;
      background: #f9f9f9;
    }
    h1 {
      color: #333;
    }
    select, input[type="number"] {
      padding: 6px;
      margin: 5px 0;
      width: 100%;
    }
    .card-select {
      margin-bottom: 20px;
    }
    .benefits-table {
      width: 100%;
      border-collapse: collapse;
      margin-bottom: 20px;
    }
    .benefits-table th, .benefits-table td {
      border: 1px solid #ccc;
      padding: 10px;
      background: #fff;
    }
    .result-box {
      border: 2px solid #ff9800;
      padding: 16px;
      background: #fff8e1;
      border-radius: 8px;
      font-size: 18px;
      font-weight: bold;
      color: #333;
      margin-top: 24px;
    }
    .calculate-btn {
      display: block;
      margin: 20px auto;
      padding: 12px 24px;
      font-size: 16px;
      background: #ff9800;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
    }
    .calculate-btn:hover {
      background: #fb8c00;
    }
    .hidden {
      display: none;
    }
  </style>
</head>
<body>

  <h1>クレジットカード特典シミュレーター</h1>

  <div class="card-select">
    <label for="cardSelect">クレジットカードを選択：</label>
    <select id="cardSelect"></select>
  </div>

  <table class="benefits-table" id="benefitsTable">
    <thead>
      <tr><th>特典名</th><th>目安金額</th><th>あなたの評価</th></tr>
    </thead>
    <tbody></tbody>
  </table>

  <button class="calculate-btn" onclick="calculate()">▶ 結果を表示</button>

  <div class="result-box hidden" id="resultBox"></div>

  <script>
    const API_URL = 'https://script.google.com/macros/s/AKfycbyaaPZ-KcL3b9VuYswFt4y2BMI_wmeZLx2zf8wh25GxfCy2WULu9b6FW2cdI0c49SzCHA/exec';
    let cardData = [];

    async function fetchCardData() {
      try {
        const res = await fetch(API_URL);
        const data = await res.json();
        cardData = data;
        populateCardOptions();
      } catch (e) {
        alert('カードデータの取得に失敗しました。');
        console.error(e);
      }
    }

    function populateCardOptions() {
      const select = document.getElementById('cardSelect');
      select.innerHTML = '<option value="">-- カードを選択 --</option>';
      cardData.forEach((card, idx) => {
        const opt = document.createElement('option');
        opt.value = idx;
        opt.textContent = card['カード名'];
        select.appendChild(opt);
      });

      select.addEventListener('change', (e) => {
        const index = e.target.value;
        if (index !== "") {
          renderBenefits(cardData[index]);
        }
      });
    }

    function renderBenefits(card) {
      const tbody = document.getElementById('benefitsTable').querySelector('tbody');
      tbody.innerHTML = '';
      for (let i = 1; i <= 10; i++) {
        const name = card[`特典${i}`];
        const base = card[`特典${i}_目安`] || 0;
        if (name) {
          const tr = document.createElement('tr');
          tr.innerHTML = `
            <td>${name}</td>
            <td>${base} 円</td>
            <td><input type="number" min="0" data-index="${i}" placeholder="金額を入力（円）"></td>
          `;
          tbody.appendChild(tr);
        }
      }
      document.getElementById('resultBox').classList.add('hidden');
    }

    function calculate() {
      const select = document.getElementById('cardSelect');
      const card = cardData[select.value];
      if (!card) return;

      const inputs = document.querySelectorAll('#benefitsTable input');
      let total = 0;
      inputs.forEach(input => {
        const val = parseInt(input.value);
        if (!isNaN(val)) total += val;
      });

      const annualFee = parseInt(card['年会費'].toString().replace(/[^\d]/g, '')) || 0;
      const diff = total - annualFee;

      const resultBox = document.getElementById('resultBox');
      resultBox.innerHTML = `
        総合評価額：<b>${total.toLocaleString()}円</b><br>
        年会費：<b>${annualFee.toLocaleString()}円</b><br>
        差額：<b style="color:${diff >= 0 ? 'green' : 'red'};">${diff.toLocaleString()}円 ${diff >= 0 ? '（お得！）' : '（損かも…）'}</b>
      `;
      resultBox.classList.remove('hidden');
    }

    fetchCardData();
  </script>

</body>
</html>
