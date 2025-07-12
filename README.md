<html lang="zh-TW">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>桌次維護公告產生器</title>
  <style>
    /* Reset & Base */
    * {
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

/* 主容器 */
.container {
  background: #fff;
  max-width: 1140px;
  width: 100%;
  margin: 0 auto;
  border-radius: 16px;
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.08);
  padding: 40px 48px 48px 48px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

/* 標題 */
h2 {
  margin: 0 0 20px 0;
  font-weight: 800;
  font-size: 1.8rem;
  color: #0078d7;
  text-align: center;
  letter-spacing: 0.5px;
}

/* Label 與欄位 */
label {
  font-weight: 600;
  margin-bottom: 8px;
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

/* 按鈕 */
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

/* Textarea */
textarea {
  resize: vertical;
  min-height: 200px;
  font-family: "Courier New", Courier, monospace;
  background: #f9fbff;
  border: 2px solid #d6deec;
  box-shadow: inset 0 2px 6px #e0eaf5;
  border-radius: 10px;
  font-size: 1rem;
}

/* Checkbox 群組 */
.checkbox-group {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-top: 8px;
}
.checkbox-group label {
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 10px;
  color: #555;
  cursor: pointer;
}
.checkbox-group input[type="checkbox"] {
  width: 18px;
  height: 18px;
}

/* 響應式 */
@media (max-width: 600px) {
  body {
    padding: 30px 16px;
  }
  .container {
    padding: 24px 20px 32px 20px;
  }
  button {
    font-size: 1rem;
  }
}

  </style>
</head>
<body>
  <main class="container" role="main" aria-label="桌次維護公告產生器">
    <h2>🎥 桌次維護公告產生器</h2>

    <div>
      <label for="date">📅 日期：</label>
      <input type="date" id="date" aria-required="true" />
    </div>

    <div>
      <label for="startTime">🕓 起始時間：</label>
      <input type="time" id="startTime" aria-required="true" />
    </div>

    <div>
      <label for="endTime">🕘 結束時間：</label>
      <input type="time" id="endTime" aria-required="true" />
    </div>

    <div>
      <label for="source">🎲 視訊源：</label>
      <select id="source" aria-required="true">
        <option value="">請選擇</option>
        <option value="BC">BC</option>
        <option value="AS">AS</option>
        <option value="MX">MX</option>
      </select>
    </div>

    <fieldset class="checkbox-group" aria-label="選擇公告類型">
      <label for="early">
        <input type="checkbox" id="early" />
        提前完成
      </label>
      <label for="extend">
        <input type="checkbox" id="extend" />
        延長維護
      </label>
    </fieldset>

    <button type="button" onclick="generateNotice()">產生公告</button>

    <div>
      <label for="output">📢 公告內容：</label>
      <textarea id="output" readonly aria-live="polite" aria-label="公告內容"></textarea>
    </div>
  </main>

  <script>
    window.onload = () => {
      const today = new Date().toISOString().split("T")[0];
      document.getElementById("date").value = today;

      ["date", "startTime", "endTime"].forEach((id) => {
        const el = document.getElementById(id);
        el.addEventListener("focus", () => el.showPicker && el.showPicker());
      });
    };

    const tableMap = {
      BC: "百家樂EU1～EU5、輪盤EU1",
      AS: "彈珠賽車AS1、百家樂AS1~AS5、骰寶AS1",
      MX: "21點百家樂MX1、百家樂MX1~MX10、輪盤MX1、龍虎鬥MX1",
    };

    function formatDate(dateStr) {
      if (!dateStr) return "（未選日期）";
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
      const source = document.getElementById("source").value;
      const early = document.getElementById("early").checked;
      const extend = document.getElementById("extend").checked;

      if (!date || !source || (!early && !extend && (!startTime || !endTime))) {
        alert("請填寫完整資訊！");
        return;
      }

      const tables = tableMap[source];
      const formattedDate = formatDate(date);
      let notice = "";

      if (early) {
        notice = `【BB視訊 - 桌次臨時維護 提前完成通知】

桌次：${tables}

請玩家重整後，即可進入遊戲

______通知您`;
      } else if (extend) {
        if (!endTime) {
          alert("請選擇結束時間！");
          return;
        }
        notice = `【BB視訊 - 桌次臨時維護 延長通知】

影響桌次：${tables}

延長至北京時間：【${formattedDate} ${endTime}】

______通知您`;
      } else {
        notice = `【BB視訊 - 桌次臨時維護通知】

影響桌次：${tables}

北京時間：【${formattedDate} ${startTime} ～ ${endTime}】

______通知您`;
      }

      document.getElementById("output").value = notice;
    }
  </script>
</body>
</html>
