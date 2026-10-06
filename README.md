<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>YRC Verified Finisher</title>

<style>
body {
  margin: 0;
  padding: 30px 15px;
  background: #111827;
  color: white;
  font-family: Arial, sans-serif;
}

.card {
  max-width: 650px;
  margin: auto;
  background: linear-gradient(145deg, #182333, #0d1420);
  border: 1px solid #64748b;
  border-radius: 18px;
  padding: 30px;
  box-sizing: border-box;
  text-align: center;
  box-shadow: 0 10px 35px rgba(0,0,0,.45);
}

.club {
  letter-spacing: 4px;
  font-size: 15px;
  font-weight: bold;
  margin-bottom: 20px;
}

.verified {
  font-size: 28px;
  font-weight: bold;
  margin-bottom: 30px;
}

.check {
  color: #4ade80;
}

.name {
  font-size: 32px;
  font-weight: bold;
  margin-bottom: 8px;
}

.event {
  font-size: 18px;
  margin-bottom: 25px;
  color: #d1d5db;
}

.grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
}

.box {
  background: #64748b;
  background: rgba(100,116,139,.45);
  border-radius: 12px;
  padding: 18px 10px;
}

.label {
  font-size: 12px;
  letter-spacing: 1px;
  color: #e5e7eb;
  margin-bottom: 8px;
}

.value {
  font-size: 20px;
  font-weight: bold;
}

.medal {
  margin-top: 18px;
  padding: 16px;
  border: 1px solid #94a3b8;
  border-radius: 12px;
  font-weight: bold;
}

.footer {
  margin-top: 30px;
  color: #9ca3af;
  font-size: 12px;
}

.error {
  color: #fca5a5;
  font-size: 18px;
  line-height: 1.5;
}

.loading {
  color: #d1d5db;
  font-size: 18px;
}
</style>
</head>

<body>

<div class="card" id="app">
  <div class="loading">Verifying YRC finisher...</div>
</div>

<script>

const CSV_URL =
"https://docs.google.com/spreadsheets/d/e/2PACX-1vSiR8BrRCuLcHm5x3PLy4_aSIFSB6rfgx53eTJylBvGlJkTqRTYzO5Qoc7MIPIcg2LBe_kDS3Pxyt0u/pub?gid=1167424338&single=true&output=csv";


function parseCSV(text) {

  const rows = [];
  let row = [];
  let cell = "";
  let insideQuotes = false;

  for (let i = 0; i < text.length; i++) {

    const char = text[i];
    const next = text[i + 1];

    if (char === '"' && insideQuotes && next === '"') {
      cell += '"';
      i++;
    }

    else if (char === '"') {
      insideQuotes = !insideQuotes;
    }

    else if (char === ',' && !insideQuotes) {
      row.push(cell);
      cell = "";
    }

    else if ((char === '\n' || char === '\r') && !insideQuotes) {

      if (char === '\r' && next === '\n') {
        i++;
      }

      row.push(cell);
      rows.push(row);
      row = [];
      cell = "";
    }

    else {
      cell += char;
    }
  }

  if (cell !== "" || row.length > 0) {
    row.push(cell);
    rows.push(row);
  }

  return rows;
}


function escapeHTML(value) {

  return String(value ?? "")
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");
}


async function verifyMedal() {

  const params = new URLSearchParams(window.location.search);
  const requestedID = params.get("id");

  const app = document.getElementById("app");

  if (!requestedID) {

    app.innerHTML = `
      <div class="club">YOKOSUKA RUCK CLUB</div>
      <div class="error">
        No Medal ID was provided.
      </div>
    `;

    return;
  }

  try {

    const response = await fetch(CSV_URL);

    if (!response.ok) {
      throw new Error("Unable to access verification data.");
    }

    const csv = await response.text();
    const rows = parseCSV(csv);

    if (rows.length < 2) {
      throw new Error("No verification records found.");
    }

    const headers = rows[0].map(h => h.trim());

    const records = rows.slice(1).map(row => {

      const record = {};

      headers.forEach((header, index) => {
        record[header] = (row[index] || "").trim();
      });

      return record;
    });

    const finisher = records.find(
      record =>
        record.MEDAL_ID.toUpperCase() === requestedID.toUpperCase()
    );

    if (!finisher) {

      app.innerHTML = `
        <div class="club">YOKOSUKA RUCK CLUB</div>

        <div class="verified">
          <span class="check">✕</span>
          MEDAL NOT FOUND
        </div>

        <div class="error">
          Medal ID:<br>
          <strong>${escapeHTML(requestedID)}</strong>
          <br><br>
          This medal could not be verified.
        </div>
      `;

      return;
    }

    app.innerHTML = `

      <div class="club">
        YOKOSUKA RUCK CLUB
      </div>

      <div class="verified">
        <span class="check">✓</span>
        VERIFIED FINISHER
      </div>

      <div class="name">
        ${escapeHTML(finisher.NAME)}
      </div>

      <div class="event">
        ${escapeHTML(finisher.EVENT)}
      </div>

      <div class="grid">

        <div class="box">
          <div class="label">DISTANCE</div>
          <div class="value">
            ${escapeHTML(finisher.DISTANCE)}
          </div>
        </div>

        <div class="box">
          <div class="label">RUCK WEIGHT</div>
          <div class="value">
            ${escapeHTML(finisher.WEIGHT)}
          </div>
        </div>

        <div class="box">
          <div class="label">FINISH TIME</div>
          <div class="value">
            ${escapeHTML(finisher.TIME)}
          </div>
        </div>

        <div class="box">
          <div class="label">COMPLETION DATE</div>
          <div class="value">
            ${escapeHTML(finisher.DATE)}
          </div>
        </div>

      </div>

      <div class="medal">
        MEDAL ID<br>
        ${escapeHTML(finisher.MEDAL_ID)}
      </div>

      <div class="footer">
        This digital finisher medal has been verified by<br>
        Yokosuka Ruck Club.
      </div>

    `;

  } catch (error) {

    app.innerHTML = `
      <div class="club">YOKOSUKA RUCK CLUB</div>

      <div class="verified">
        <span class="check">!</span>
        VERIFICATION ERROR
      </div>

      <div class="error">
        ${escapeHTML(error.message)}
      </div>
    `;
  }
}


verifyMedal();

</script>

</body>
</html>
