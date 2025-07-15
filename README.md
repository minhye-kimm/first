[Uploading random.html…<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <title>팀 응모 및 추첨 프로그램</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
  <style>
    body {
      font-family: 'Arial', sans-serif;
      margin: 0;
      padding: 0;
      background: #f4f4f4;
      display: flex;
      justify-content: center;
    }

    .container {
      max-width: 700px;
      width: 100%;
      background: #fff;
      padding: 30px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
      margin: 20px;
      border-radius: 10px;
    }

    h2 {
      text-align: center;
      margin-bottom: 20px;
    }

    .form-section {
      display: flex;
      flex-direction: column;
      gap: 15px;
    }

    label {
      font-size: 1.1rem;
    }

    input {
      width: 100%;
      padding: 10px;
      font-size: 1rem;
      margin-top: 5px;
      border: 1px solid #ccc;
      border-radius: 6px;
    }

    button {
      padding: 12px;
      font-size: 1rem;
      border: none;
      border-radius: 6px;
      background-color: #007BFF;
      color: white;
      cursor: pointer;
      transition: background 0.3s;
    }

    button:hover {
      background-color: #0056b3;
    }

    .button-group {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-top: 20px;
      justify-content: center;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }

    th, td {
      border: 1px solid #aaa;
      padding: 10px;
      text-align: center;
    }

    #admin-controls {
      display: none;
      flex-direction: column;
      gap: 10px;
      margin-top: 20px;
    }

    @media (max-width: 768px) {
      .container {
        padding: 20px;
      }

      button {
        font-size: 0.95rem;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <h2>상품 추첨 응모</h2>

    <div class="form-section">
      <label>팀 번호:
        <input type="number" id="team-number" />
      </label>

      <label>이름 1:
        <input type="text" class="member-name" />
      </label>

      <label>이름 2:
        <input type="text" class="member-name" />
      </label>

      <label>이름 3:
        <input type="text" class="member-name" />
      </label>

      <label>이름 4:
        <input type="text" class="member-name" />
      </label>

      <button onclick="submitEntry()">응모하기</button>
    </div>

    <hr style="margin: 30px 0;">

    <div class="button-group">
      <button onclick="toggleAdminMode()">관리자 모드</button>
    </div>

    <div id="admin-controls" class="button-group">
      <button onclick="endEntry()">응모 종료하기</button>
      <button onclick="drawWinners()">추첨하기</button>
      <button onclick="downloadExcel()">엑셀로 저장</button>
      <button onclick="showEntries()">응모 현황 보기</button>
      <button onclick="resetAll()">전체 초기화</button>
    </div>

    <div id="winners"></div>
    <div id="entry-list"></div>
  </div>

  <script>
    let entries = [];
    let entryClosed = false;
    let drawDone = false;
    const adminPassword = "1925";
    let adminVisible = false;

    function toggleAdminMode() {
      if (adminVisible) {
        adminVisible = false;
        document.getElementById("admin-controls").style.display = "none";
        return;
      }

      const password = prompt("관리자 비밀번호를 입력하세요:");
      if (password === adminPassword) {
        adminVisible = true;
        document.getElementById("admin-controls").style.display = "flex";
      } else {
        alert("비밀번호가 틀렸습니다.");
      }
    }

    function submitEntry() {
      if (entryClosed) {
        alert("응모가 종료되었습니다.");
        return;
      }

      const teamNumber = document.getElementById("team-number").value.trim();
      const names = Array.from(document.getElementsByClassName("member-name"))
        .map(input => input.value.trim())
        .filter(name => name !== "");

      if (!teamNumber || names.length === 0) {
        alert("팀 번호와 이름을 최소 1명 이상 입력해주세요.");
        return;
      }

      const isDuplicate = entries.some(entry => entry.team === teamNumber);
      if (isDuplicate) {
        alert("이미 등록된 팀 번호입니다. 다른 팀 번호를 입력해주세요.");
        return;
      }

      const confirmMessage = `입력하신 팀: ${teamNumber}팀\n이름: ${names.join(", ")}\n\n위 내용이 맞습니까?`;
      if (!confirm(confirmMessage)) return;

      entries.push({ team: teamNumber, members: names });
      alert("응모가 완료되었습니다. God Bless You :)");
      resetForm();
    }

    function resetForm() {
      document.getElementById("team-number").value = "";
      document.querySelectorAll(".member-name").forEach(input => input.value = "");
    }

    function endEntry() {
      if (entryClosed) {
        alert("이미 응모가 종료되었습니다.");
        return;
      }
      entryClosed = true;
      alert("응모가 종료되었습니다.");
    }

    function drawWinners() {
      if (drawDone) {
        alert("이미 추첨이 완료되었습니다.");
        return;
      }

      if (entries.length < 12) {
        alert("12팀 이상 응모되어야 추첨이 가능합니다.");
        return;
      }

      const shuffled = [...entries].sort(() => Math.random() - 0.5);
      const winners = shuffled.slice(0, 12);

      let html = "<h3>당첨 팀 (총 12팀)</h3><table><tr><th>팀 번호</th><th>이름</th></tr>";
      winners.forEach(entry => {
        html += `<tr><td>${entry.team}</td><td>${entry.members.join(", ")}</td></tr>`;
      });
      html += "</table>";
      document.getElementById("winners").innerHTML = html;

      window.winnerData = winners;
      drawDone = true;
    }

    function downloadExcel() {
      if (!window.winnerData || window.winnerData.length === 0) {
        alert("먼저 추첨을 진행해주세요.");
        return;
      }

      const wsData = [["팀 번호", "이름"]];
      window.winnerData.forEach(entry => {
        wsData.push([entry.team, entry.members.join(", ")]);
      });

      const worksheet = XLSX.utils.aoa_to_sheet(wsData);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, "Winners");

      XLSX.writeFile(workbook, "당첨팀.xlsx");
    }

    function showEntries() {
      if (entries.length === 0) {
        alert("아직 등록된 팀이 없습니다.");
        return;
      }

      let html = "<h3>응모 현황</h3><table><tr><th>팀 번호</th><th>이름</th></tr>";
      entries.forEach(entry => {
        html += `<tr><td>${entry.team}</td><td>${entry.members.join(", ")}</td></tr>`;
      });
      html += "</table>";
      document.getElementById("entry-list").innerHTML = html;
    }

    function resetAll() {
      const confirmed = confirm("정말 모든 정보를 초기화하시겠습니까?");
      if (!confirmed) return;

      entries = [];
      entryClosed = false;
      drawDone = false;
      window.winnerData = [];
      document.getElementById("entry-list").innerHTML = "";
      document.getElementById("winners").innerHTML = "";
      alert("모든 데이터가 초기화되었습니다.");
    }
  </script>
</body>
</html>
]()
