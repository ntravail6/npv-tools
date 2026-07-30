<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>社有車 運転日報入力システム</title>
  <style>
    /* Meiryo UI を最優先指定 */
    body {
      font-family: "Meiryo UI", "Meiryo", "メイリオ", sans-serif;
      background-color: #f4f6f9;
      color: #333;
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      min-height: 100vh;
    }

    .app-container {
      width: 100%;
      max-width: 480px;
      background: #ffffff;
      min-height: 100vh;
      box-shadow: 0 4px 20px rgba(0,0,0,0.08);
      display: flex;
      flex-direction: column;
    }

    header {
      background-color: #0f4c81;
      color: #ffffff;
      padding: 16px 20px;
      text-align: center;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }

    header h1 {
      margin: 0;
      font-size: 18px;
      font-weight: bold;
      letter-spacing: 0.5px;
    }

    .content {
      padding: 20px;
      flex: 1;
    }

    /* プログレスバー */
    .step-indicator {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 24px;
      position: relative;
    }

    .step-indicator::before {
      content: '';
      position: absolute;
      top: 50%;
      left: 0;
      right: 0;
      height: 2px;
      background: #e0e0e0;
      z-index: 1;
      transform: translateY(-50%);
    }

    .step-badge {
      width: 32px;
      height: 32px;
      border-radius: 50%;
      background: #e0e0e0;
      color: #666;
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: bold;
      font-size: 14px;
      z-index: 2;
      transition: all 0.3s ease;
    }

    .step-badge.active {
      background: #0f4c81;
      color: #ffffff;
    }

    .step-badge.completed {
      background: #28a745;
      color: #ffffff;
    }

    .step-title {
      font-size: 16px;
      font-weight: bold;
      color: #0f4c81;
      margin-bottom: 16px;
      border-bottom: 2px solid #0f4c81;
      padding-bottom: 6px;
    }

    /* ステップ制御 */
    .step-page {
      display: none;
    }

    .step-page.active {
      display: block;
      animation: fadeIn 0.3s ease-in-out;
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(6px); }
      to { opacity: 1; transform: translateY(0); }
    }

    /* 車両選択グリッド */
    .vehicle-grid {
      display: grid;
      grid-template-columns: 1fr;
      gap: 12px;
    }

    .vehicle-card {
      background: #f8f9fa;
      border: 2px solid #e9ecef;
      border-radius: 10px;
      padding: 16px;
      cursor: pointer;
      transition: all 0.2s ease;
      display: flex;
      align-items: center;
      justify-content: space-between;
    }

    .vehicle-card:hover, .vehicle-card:active {
      border-color: #0f4c81;
      background: #eef5fc;
      transform: translateY(-2px);
    }

    .vehicle-info .num {
      font-size: 16px;
      font-weight: bold;
      color: #111;
    }

    .vehicle-info .name {
      font-size: 13px;
      color: #666;
      margin-top: 4px;
    }

    .vehicle-odo {
      font-size: 12px;
      background: #e2e8f0;
      padding: 4px 8px;
      border-radius: 4px;
      color: #475569;
    }

    /* 選択済み車両のミニ表示 */
    .selected-vehicle-banner {
      background: #eef5fc;
      border: 1px solid #b6d4f3;
      border-radius: 8px;
      padding: 12px 16px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
    }

    .selected-vehicle-banner .label {
      font-size: 12px;
      color: #555;
    }

    .selected-vehicle-banner .val {
      font-size: 15px;
      font-weight: bold;
      color: #0f4c81;
    }

    .btn-change {
      background: none;
      border: 1px solid #0f4c81;
      color: #0f4c81;
      padding: 4px 10px;
      border-radius: 4px;
      font-size: 12px;
      cursor: pointer;
    }

    /* フォーム要素 */
    .form-group {
      margin-bottom: 18px;
    }

    .form-group label {
      display: block;
      font-size: 13px;
      font-weight: bold;
      color: #444;
      margin-bottom: 6px;
    }

    .form-group label .req {
      color: #dc3545;
      margin-left: 4px;
    }

    .form-control {
      width: 100%;
      padding: 12px;
      font-size: 15px;
      font-family: "Meiryo UI", sans-serif;
      border: 1px solid #ccc;
      border-radius: 6px;
      box-sizing: border-box;
      background: #fff;
    }

    .form-control:focus {
      border-color: #0f4c81;
      outline: none;
      box-shadow: 0 0 0 3px rgba(15, 76, 129, 0.15);
    }

    .form-control[readonly] {
      background-color: #e9ecef;
      color: #495057;
      cursor: not-allowed;
    }

    /* 自動計算用の強調スタイル */
    .calc-highlight {
      background-color: #f0fdf4 !important;
      border-color: #86efac !important;
      font-weight: bold;
      color: #166534;
      font-size: 18px;
    }

    .unit-input-group {
      position: relative;
      display: flex;
      align-items: center;
    }

    .unit-input-group .form-control {
      padding-right: 40px;
    }

    .unit-input-group .unit {
      position: absolute;
      right: 12px;
      font-size: 14px;
      color: #666;
      pointer-events: none;
    }

    /* ボタン */
    .btn-submit {
      width: 100%;
      background: #0f4c81;
      color: #fff;
      border: none;
      padding: 14px;
      font-size: 16px;
      font-weight: bold;
      font-family: "Meiryo UI", sans-serif;
      border-radius: 8px;
      cursor: pointer;
      box-shadow: 0 4px 6px rgba(15, 76, 129, 0.2);
      transition: background 0.2s;
      margin-top: 10px;
    }

    .btn-submit:hover {
      background: #0b3860;
    }

    .btn-submit:disabled {
      background: #ccc;
      cursor: not-allowed;
      box-shadow: none;
    }

    /* 送信完了オーバーレイ */
    .modal {
      display: none;
      position: fixed;
      top: 0; left: 0; width: 100%; height: 100%;
      background: rgba(0,0,0,0.5);
      justify-content: center;
      align-items: center;
      z-index: 100;
    }

    .modal-content {
      background: #fff;
      padding: 24px;
      border-radius: 12px;
      text-align: center;
      max-width: 320px;
      width: 80%;
    }

    .modal-icon {
      font-size: 48px;
      color: #28a745;
      margin-bottom: 12px;
    }
  </style>
</head>
<body>

<div class="app-container">
  <header>
    <h1>社有車 運転日報入力</h1>
  </header>

  <div class="content">
    <!-- ステップインジケーター -->
    <div class="step-indicator">
      <div class="step-badge active" id="badge-step1">1</div>
      <div class="step-badge" id="badge-step2">2</div>
    </div>

    <!-- STEP 1: 車両選択画面 -->
    <div class="step-page active" id="step1">
      <div class="step-title">① 使用する車両を選択してください</div>
      <div class="vehicle-grid" id="vehicleContainer">
        <!-- JSで動的生成 -->
      </div>
    </div>

    <!-- STEP 2: 日報入力画面 -->
    <div class="step-page" id="step2">
      <!-- 選択中車両の表示 -->
      <div class="selected-vehicle-banner">
        <div>
          <div class="label">選択中の車両</div>
          <div class="val" id="selectedVehicleName">-</div>
        </div>
        <button type="button" class="btn-change" onclick="goToStep(1)">車両変更</button>
      </div>

      <form id="dailyReportForm" onsubmit="handleFormSubmit(event)">
        <!-- ドライバー名 -->
        <div class="form-group">
          <label for="driverName">ドライバー名<span class="req">*</span></label>
          <input type="text" id="driverName" class="form-control" placeholder="例: 山田 太郎" required>
        </div>

        <!-- 日付・時刻 -->
        <div class="form-group">
          <label for="reportDateTime">日時<span class="req">*</span></label>
          <input type="datetime-local" id="reportDateTime" class="form-control" required>
        </div>

        <!-- 直前の走行距離 -->
        <div class="form-group">
          <label for="prevOdometer">直前の走行距離 (自動取得)</label>
          <div class="unit-input-group">
            <input type="number" id="prevOdometer" class="form-control" readonly>
            <span class="unit">km</span>
          </div>
        </div>

        <!-- 最終走行距離 -->
        <div class="form-group">
          <label for="finalOdometer">最終走行距離<span class="req">*</span></label>
          <div class="unit-input-group">
            <input type="number" id="finalOdometer" class="form-control" placeholder="数字を入力" step="1" min="0" required oninput="calculateDistance()">
            <span class="unit">km</span>
          </div>
        </div>

        <!-- 本日の走行距離 -->
        <div class="form-group">
          <label for="todayDistance">本日の走行距離 (自動計算)</label>
          <div class="unit-input-group">
            <input type="number" id="todayDistance" class="form-control calc-highlight" readonly placeholder="0">
            <span class="unit">km</span>
          </div>
        </div>

        <!-- 目的地・業務内容 -->
        <div class="form-group">
          <label for="destination">行き先・主な業務内容</label>
          <input type="text" id="destination" class="form-control" placeholder="例: △△株式会社様 訪問">
        </div>

        <!-- 点検・アルコールチェック -->
        <div class="form-group">
          <label for="inspection">アルコールチェック・日常点検</label>
          <select id="inspection" class="form-control">
            <option value="異常なし">異常なし（酒気帯びなし・点検済）</option>
            <option value="要確認">要確認・補修あり</option>
          </select>
        </div>

        <!-- 送信ボタン -->
        <button type="submit" class="btn-submit" id="submitBtn">日報を送信する</button>
      </form>
    </div>
  </div>
</div>

<!-- 完了モーダル -->
<div class="modal" id="successModal">
  <div class="modal-content">
    <div class="modal-icon">✓</div>
    <h3 style="margin: 0 0 10px 0;">送信完了</h3>
    <p style="font-size: 14px; color: #666; margin-bottom: 20px;">運転日報が正常に送信され、Excelに記録されました。</p>
    <button class="btn-submit" onclick="resetForm()">新しい日報を入力する</button>
  </div>
</div>

<script>
  // ----------------------------------------------------
  // 設定: Power AutomateのHTTP POST用Webhook URL
  // ----------------------------------------------------
  const POWER_AUTOMATE_WEBHOOK_URL = ""; 

  // サンプル車両マスターデータ（5台分）
  const vehicleList = [
    { id: 'car1', number: '品川 501 さ 10-01', name: 'トヨタ アクア（営業1号）', lastOdo: 32450 },
    { id: 'car2', number: '品川 400 た 20-02', name: 'トヨタ プロボックス（営業2号）', lastOdo: 58120 },
    { id: 'car3', number: '品川 300 な 30-03', name: '日産 セレナ（役員・送迎用）', lastOdo: 15300 },
    { id: 'car4', number: '品川 480 ら 40-04', name: 'ダイハツ ハイゼット（現場用）', lastOdo: 82900 },
    { id: 'car5', number: '品川 500 ま 50-05', name: 'ホンダ フィット（本社共有）', lastOdo: 21800 }
  ];

  let selectedVehicle = null;

  // 初期化処理
  window.addEventListener('DOMContentLoaded', () => {
    renderVehicleList();
    setCurrentDateTime();
  });

  // 現在の日時を "YYYY-MM-DDThh:mm" 形式でデフォルト設定
  function setCurrentDateTime() {
    const now = new Date();
    const year = now.getFullYear();
    const month = String(now.getMonth() + 1).padStart(2, '0');
    const day = String(now.getDate()).padStart(2, '0');
    const hours = String(now.getHours()).padStart(2, '0');
    const minutes = String(now.getMinutes()).padStart(2, '0');
    
    const formatted = `${year}-${month}-${day}T${hours}:${minutes}`;
    document.getElementById('reportDateTime').value = formatted;
  }

  // 車両リストの描画
  function renderVehicleList() {
    const container = document.getElementById('vehicleContainer');
    container.innerHTML = '';

    vehicleList.forEach(car => {
      const card = document.createElement('div');
      card.className = 'vehicle-card';
      card.onclick = () => selectVehicle(car);

      card.innerHTML = `
        <div class="vehicle-info">
          <div class="num">${car.number}</div>
          <div class="name">${car.name}</div>
        </div>
        <div class="vehicle-odo">直前: ${car.lastOdo.toLocaleString()} km</div>
      `;
      container.appendChild(card);
    });
  }

  // 車両選択時の処理
  function selectVehicle(car) {
    selectedVehicle = car;
    
    document.getElementById('selectedVehicleName').innerText = `${car.number} (${car.name})`;
    document.getElementById('prevOdometer').value = car.lastOdo;
    document.getElementById('finalOdometer').value = '';
    document.getElementById('todayDistance').value = '';

    goToStep(2);
  }

  // ステップ切り替え
  function goToStep(stepNumber) {
    document.getElementById('step1').classList.remove('active');
    document.getElementById('step2').classList.remove('active');

    document.getElementById('badge-step1').classList.remove('active', 'completed');
    document.getElementById('badge-step2').classList.remove('active', 'completed');

    if (stepNumber === 1) {
      document.getElementById('step1').classList.add('active');
      document.getElementById('badge-step1').classList.add('active');
    } else if (stepNumber === 2) {
      document.getElementById('step2').classList.add('active');
      document.getElementById('badge-step1').classList.add('completed');
      document.getElementById('badge-step2').classList.add('active');
    }
  }

  // 走行距離の自動計算
  function calculateDistance() {
    const prev = parseFloat(document.getElementById('prevOdometer').value) || 0;
    const finalVal = parseFloat(document.getElementById('finalOdometer').value);

    const todayDistanceInput = document.getElementById('todayDistance');

    if (!isNaN(finalVal)) {
      const diff = finalVal - prev;
      if (diff >= 0) {
        todayDistanceInput.value = diff;
      } else {
        todayDistanceInput.value = "エラー(直前より小さい)";
      }
    } else {
      todayDistanceInput.value = '';
    }
  }

  // フォーム送信処理
  async function handleFormSubmit(event) {
    event.preventDefault();

    const submitBtn = document.getElementById('submitBtn');
    const finalVal = parseFloat(document.getElementById('finalOdometer').value);
    const prevVal = parseFloat(document.getElementById('prevOdometer').value);

    if (finalVal < prevVal) {
      alert('最終走行距離は直前の走行距離（' + prevVal + 'km）以上の数値を入力してください。');
      return;
    }

    const formData = {
      vehicleNumber: selectedVehicle.number,
      vehicleName: selectedVehicle.name,
      driverName: document.getElementById('driverName').value,
      dateTime: document.getElementById('reportDateTime').value,
      prevOdometer: prevVal,
      finalOdometer: finalVal,
      todayDistance: parseFloat(document.getElementById('todayDistance').value) || 0,
      destination: document.getElementById('destination').value,
      inspection: document.getElementById('inspection').value,
      submittedAt: new Date().toISOString()
    };

    submitBtn.disabled = true;
    submitBtn.innerText = '送信中...';

    try {
      if (POWER_AUTOMATE_WEBHOOK_URL) {
        const response = await fetch(POWER_AUTOMATE_WEBHOOK_URL, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(formData)
        });

        if (!response.ok) throw new Error('送信エラーが発生しました');
      } else {
        console.log('送信データ (デモ):', formData);
        await new Promise(resolve => setTimeout(resolve, 1000));
      }

      selectedVehicle.lastOdo = finalVal;
      renderVehicleList();

      document.getElementById('successModal').style.display = 'flex';

    } catch (err) {
      alert('送信に失敗しました: ' + err.message);
    } finally {
      submitBtn.disabled = false;
      submitBtn.innerText = '日報を送信する';
    }
  }

  function resetForm() {
    document.getElementById('successModal').style.display = 'none';
    document.getElementById('dailyReportForm').reset();
    setCurrentDateTime();
    goToStep(1);
  }
</script>

</body>
</html>
