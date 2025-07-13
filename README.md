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
      max-width: 600px;
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
      font-size: 1.8rem;
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
        </select>
      </div>
    </div>
    <div id="tableActions" class="row">
      <button type="button" id="selectAllBtn">全桌次</button>
      <button type="button" id="deselectAllBtn">取消全選</button>
    </div>
    <div id="tableCheckboxes" class="checkbox-group"></div>
    <div class="checkbox-group">
      <label><input type="checkbox" id="early" /> 提前完成</label>
      <label><input type="checkbox" id="extend" /> 延長維護</label>
    </div>
    <div class="bottom-buttons">
      <button onclick="generateNotice()">產生公告</button>
      <button class="btn-copy" onclick="copyNotice()">📋 複製公告</button>
    </div>
    <div>
      <label for="output">📢 公告內容：</label>
      <textarea id="output" readonly></textarea>
    </div>
  </main>
  <script>
    const tableOptions = {
      BC: ["百家樂EU1", "百家樂EU2", "百家樂EU3"],
      AS: ["百家樂AS1", "百家樂AS2", "百家樂AS3"],
      MX: ["百家樂MX1", "百家樂MX2", "百家樂MX3"]
    };

    window.onload = () => {
      const today = new Date().toISOString().split("T")[0];
      document.getElementById("date").value = today;
    };

    document.getElementById("source").addEventListener("change", function () {
      const source = this.value;
      const tableDiv = document.getElementById("tableCheckboxes");
      const actionDiv = document.getElementById("tableActions");
      const selectAllBtn = document.getElementById("selectAllBtn");
      tableDiv.innerHTML = "";
      actionDiv.style.display = "none";

      let allTables = [];
      if (source === "all") {
        Object.values(tableOptions).forEach(list => allTables.push(...list));
        selectAllBtn.textContent = `全部 全桌次`;
        actionDiv.style.display = "flex";
      } else if (tableOptions[source]) {
        allTables = tableOptions[source];
        selectAllBtn.textContent = `${source} 全桌次`;
        actionDiv.style.display = "flex";
      }

      allTables.forEach((name, index) => {
        const checkboxId = `table_${index}`;
        const label = document.createElement("label");
        label.innerHTML = `<input type="checkbox" id="${checkboxId}" value="${name}"> ${name}`;
        tableDiv.appendChild(label);
      });
    });

    document.getElementById("selectAllBtn").addEventListener("click", () => {
      document.querySelectorAll("#tableCheckboxes input[type='checkbox']").forEach(cb => cb.checked = true);
    });

    document.getElementById("deselectAllBtn").addEventListener("click", () => {
      document.querySelectorAll("#tableCheckboxes input[type='checkbox']").forEach(cb => cb.checked = false);
    });

    function getSelectedTables() {
      return Array.from(document.querySelectorAll("#tableCheckboxes input[type='checkbox']:checked"))
        .map(cb => cb.value);
    }

    function formatDate(dateStr) {
      const days = ["日", "一", "二", "三", "四", "五", "六"];
      const date = new Date(dateStr);
      const mm = String(date.getMonth() + 1).padStart(2, "0");
      const dd = String(date.getDate()).padStart(2, "0");
      const day = days[date.getDay()];
      return `${mm}/${dd}(${day})`;
    }

    function generateNotice() {
      const date = document.getElementById("date").value;
      const startTime = document.getElementById("startTime").value;
      const endTime = document.getElementById("endTime").value;
      const early = document.getElementById("early").checked;
      const extend = document.getElementById("extend").checked;
      const tables = getSelectedTables();

      if (!date || (!early && !extend && (!startTime || !endTime))) {
        alert("請填寫完整資訊！");
        return;
      }

      const formattedDate = formatDate(date);
      const tableText = tables.length > 0 ? tables.join("、") : "(未選取桌次)";
      let notice = "";

      if (early) {
        notice = `【BB視訊 - 桌次臨時維護 提前完成通知】\n\n桌次：${tableText}\n\n請玩家重整後，即可進入遊戲\n\n______通知您`;
      } else if (extend) {
        notice = `【BB視訊 - 桌次臨時維護 延長通知】\n\n影響桌次：${tableText}\n\n延長至北京時間：【${formattedDate} ${endTime}】\n\n______通知您`;
      } else {
        notice = `【BB視訊 - 桌次臨時維護通知】\n\n影響桌次：${tableText}\n\n北京時間：【${formattedDate} ${startTime} ～ ${endTime}】\n\n______通知您`;
      }

      document.getElementById("output").value = notice;
    }

    function copyNotice() {
      const output = document.getElementById("output");
      output.select();
      output.setSelectionRange(0, 99999);
      document.execCommand("copy");
      alert("已複製公告內容！");
    }
  </script>
</body>
</html>
