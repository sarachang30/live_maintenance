<html lang="zh-TW">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>桌次維護公告產生器</title>
  <style>
    * {
      box-sizing: border-box;
    }
    body {
      margin: 0;
      background: linear-gradient(to bottom, #eaf3fb, #f6f9fc);
      font-family: "Noto Sans TC", "Microsoft JhengHei", Arial, sans-serif;
      color: #333;
      display: flex;
      justify-content: center;
      padding: 60px 24px;
      min-height: 100vh;
    }
    .container {
      background: #fff;
      max-width: 900px;
      width: 100%;
      margin: 0 auto;
      border-radius: 16px;
      box-shadow: 0 12px 32px rgba(0, 0, 0, 0.08);
      padding: 40px 48px;
      display: flex;
      flex-direction: column;
      gap: 24px;
    }
    h2 {
      margin: 0;
      font-weight: 800;
      font-size: 2rem;
      color: #0078d7;
      text-align: center;
      letter-spacing: 0.5px;
    }
    label {
      font-weight: 600;
      margin-bottom: 6px;
      display: block;
      color: #444;
      user-select: none;
    }
    input[type="date"],
    input[type="time"],
    select,
    textarea {
      width: 100%;
      padding: 14px 18px;
      border-radius: 10px;
      border: 2px solid #d6deec;
      font-size: 1.05rem;
      background: #fff;
      font-family: inherit;
      color: #222;
      transition: 0.25s ease;
    }
    input:focus,
    select:focus,
    textarea:focus {
      outline: none;
      border-color: #0078d7;
      box-shadow: 0 0 10px #0078d77a;
    }
    button {
      padding: 14px 24px;
      background: #0078d7;
      color: white;
      font-size: 1.1rem;
      font-weight: bold;
      border: none;
      border-radius: 10px;
      cursor: pointer;
      transition: background-color 0.25s ease;
    }
    button:hover {
      background: #0060b5;
    }
    button:active {
      background: #004c8e;
    }
    .btn-copy {
      background: #e91e63;
    }
    .btn-copy:hover {
      background: #c2185b;
    }
    textarea {
      resize: vertical;
      min-height: 400px;
      font-family: "Courier New", Courier, monospace;
      background: #f9fbff;
      border: 2px solid #d6deec;
      box-shadow: inset 0 2px 6px #e0eaf5;
      border-radius: 10px;
      font-size: 1rem;
    }
    .row {
      display: flex;
      gap: 16px;
      width: 100%;
      flex-wrap: wrap;
    }
    .row.half > div {
      flex: 1;
    }
    .checkbox-group {
      display: flex;
      gap: 12px;
      flex-wrap: wrap;
      user-select: none;
    }
    .checkbox-group label {
      font-weight: 600;
      color: #555;
      display: flex;
      align-items: center;
      padding: 6px 10px;
      background: #f1f4f8;
      border-radius: 8px;
      border: 1px solid #ccd4e0;
      cursor: pointer;
    }
    .checkbox-group input[type="checkbox"] {
      margin-right: 6px;
    }
    #tableActions {
      display: none;
      gap: 10px;
      margin-bottom: 8px;
    }
    .bottom-buttons {
      display: flex;
      gap: 12px;
      justify-content: center;
    }
    @media (max-width: 480px) {
      .row,
      .checkbox-group,
      .bottom-buttons,
      #tableActions {
        flex-direction: column;
      }
    }
  </style>
</head>
<body>
  <main class="container">
    <h2>🎥 桌次維護公告產生器</h2>
    <div class="row">
      <div>
        <label for="date">📅 日期：</label>
        <input type="date" id="date" />
      </div>
    </div>
    <div class="row half">
      <div>
        <label for="startTime">🕓 起始時間：</label>
        <input type="time" id="startTime" />
      </div>
      <div>
        <label for="endTime">🕘 結束時間：</label>
        <input type="time" id="endTime" />
      </div>
    </div>
    <div class="row">
      <div>
        <label for="source">🎲 視訊源：</label>
        <select id="source">
          <option value="">請選擇</option>
          <option value="all">全部</option>
          <option value="BC">BC</option>
          <option value="AS">AS</option>
          <option value="MX">MX</option>
          <option value="RB">RB</option>
        </select>
      </div>
    </div>
    <div id="tableActions" class="row">
      <button type="button" id="selectAllBtn">全桌次</button>
      <button type="button" id="deselectAllBtn">取消全選</button>
    </div>
    <div id="tableCheckboxes" class="checkbox-group"></div>
    <div>
      <label>⚙️ 維護情境：</label>
      <div class="checkbox-group">
        <label><input type="checkbox" id="early" /> 提前完成</label>
        <label><input type="checkbox" id="extend" /> 延長維護</label>
      </div>
    </div>
    <div>
      <label for="output">📢 公告內容：</label>
      <textarea id="output" readonly></textarea>
    </div>
    <div class="bottom-buttons">
      <button onclick="generateNotice()">產生公告</button>
      <button class="btn-copy" onclick="copyNotice()">📋 複製公告</button>
    </div>
  </main>
  <script>
    const tableOptions = {
      BC: ["百家樂EU1", "百家樂EU2", "百家樂EU3", "百家樂EU4", "百家樂EU5", "輪盤EU1"],
      AS: ["百家樂AS1", "百家樂AS2", "百家樂AS3", "百家樂AS4", "百家樂AS5", "骰寶AS1", "彈珠賽車AS1"],
      MX: ["百家樂MX1", "百家樂MX2", "百家樂MX3", "百家樂MX4", "百家樂MX5", "百家樂MX6", "百家樂MX7", "百家樂MX8", "百家樂MX9", "百家樂MX10", "龍虎鬥MX1", "輪盤MX1", "21點百家樂MX1"],
      RB: ["百家樂RB1", "百家樂RB2", "百家樂RB3", "百家樂RB4", "百家樂RB5", "百家樂RB6", "百家樂RB7", "百家樂RB8", "百家樂RB9", "百家樂RB10"]
    };
  </script>
</body>
</html>
