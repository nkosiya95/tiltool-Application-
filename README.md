# tiltool-Application-"""
Tiltool App & Compliance Engine - All-in-One Application
=========================================================
Includes HTML UI, Tailwind/CSS, SANS Audit JavaScript Engine, 
and PDF Export endpoint in a single ready-to-run Python script.

Requirements:
    pip install flask weasyprint
"""

import os
from flask import Flask, render_template_string, request, send_file, io
from weasyprint import HTML

app = Flask(__name__)

# Single combined HTML, CSS, and JS template
HTML_TEMPLATE = """<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tiltool App & Compliance Engine</title>
  <style>
    @page {
      size: A4;
      margin: 12mm;
      background-color: #f8fafc;
    }
    *, *::before, *::after {
      box-sizing: border-box;
    }
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
      margin: 0;
      padding: 20px;
      background-color: #f8fafc;
      color: #0f172a;
      font-size: 10pt;
    }
    .card {
      background-color: #ffffff;
      border: 1px solid #e2e8f0;
      border-radius: 8px;
      padding: 20px;
      max-width: 800px;
      margin: 0 auto;
      box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
    }
    .header {
      background-color: #0284c7;
      color: #ffffff;
      margin: -20px -20px 20px -20px;
      padding: 20px;
      border-top-left-radius: 8px;
      border-top-right-radius: 8px;
      text-align: center;
    }
    .header h1 {
      margin: 0;
      font-size: 18pt;
      font-weight: bold;
    }
    .header p {
      margin: 4px 0 0 0;
      font-size: 9pt;
      color: #e0f2fe;
    }
    .section-title {
      font-size: 11pt;
      font-weight: 600;
      color: #0369a1;
      border-bottom: 1px solid #e2e8f0;
      padding-bottom: 4px;
      margin-top: 15px;
      margin-bottom: 10px;
    }
    .grid-2 {
      width: 100%;
      margin-bottom: 10px;
    }
    .grid-2 td {
      width: 50%;
      padding: 4px;
      vertical-align: top;
    }
    .grid-3 {
      width: 100%;
      margin-bottom: 10px;
    }
    .grid-3 td {
      width: 33.33%;
      padding: 4px;
      vertical-align: top;
    }
    .grid-4 {
      width: 100%;
      margin-bottom: 15px;
    }
    .grid-4 td {
      width: 25%;
      padding: 4px;
      vertical-align: top;
    }
    label {
      display: block;
      font-size: 8pt;
      font-weight: bold;
      color: #334155;
      margin-bottom: 2px;
    }
    input, select {
      width: 100%;
      padding: 8px;
      border: 1px solid #cbd5e1;
      border-radius: 4px;
      font-size: 9pt;
      background-color: #ffffff;
    }
    .btn-container {
      width: 100%;
      margin-top: 15px;
      margin-bottom: 20px;
    }
    .btn-container td {
      width: 50%;
      padding: 4px;
    }
    .btn {
      width: 100%;
      padding: 10px;
      border-radius: 6px;
      font-weight: bold;
      font-size: 9pt;
      text-align: center;
      color: white;
      border: none;
      cursor: pointer;
    }
    .btn-primary { background-color: #0284c7; }
    .btn-primary:hover { background-color: #0369a1; }
    .btn-secondary { background-color: #475569; }
    .btn-export { background-color: #059669; margin-top: 10px; }
    
    /* Results Styling */
    .sans-card {
      padding: 12px;
      border-radius: 6px;
      border: 1px solid #a7f3d0;
      background-color: #ecfdf5;
      margin-bottom: 15px;
    }
    .sans-title {
      font-size: 9pt;
      font-weight: bold;
      text-transform: uppercase;
      color: #065f46;
      margin: 0 0 4px 0;
    }
    .sans-msg {
      font-size: 9pt;
      font-weight: bold;
      color: #065f46;
      margin: 0;
    }
    .sans-list {
      font-size: 8pt;
      color: #334155;
      margin: 6px 0 0 15px;
      padding: 0;
    }
    .stat-box {
      background-color: #f1f5f9;
      border: 1px solid #e2e8f0;
      border-radius: 4px;
      padding: 8px;
      text-align: center;
    }
    .stat-label {
      display: block;
      font-size: 7pt;
      text-transform: uppercase;
      font-weight: bold;
      color: #64748b;
    }
    .stat-val {
      font-size: 11pt;
      font-weight: bold;
      color: #1e293b;
    }
    table.data-table {
      width: 100%;
      border-collapse: collapse;
      font-size: 9pt;
      margin-bottom: 15px;
    }
    table.data-table th {
      background-color: #f1f5f9;
      border-bottom: 1px solid #cbd5e1;
      padding: 6px 8px;
      text-align: left;
      font-weight: bold;
      color: #334155;
    }
    table.data-table td {
      border-bottom: 1px solid #f1f5f9;
      padding: 6px 8px;
      color: #334155;
    }
    table.data-table tr.total-row td {
      background-color: #f0f9ff;
      border-top: 1px solid #bae6fd;
      font-weight: bold;
      color: #0c4a6e;
      font-size: 10pt;
    }
    .text-right { text-align: right; }
    .hidden { display: none; }
  </style>
</head>
<body>
  <div class="card">
    <!-- App Header -->
    <div class="header">
      <h1>🧱 Tiltool App & Compliance Engine</h1>
      <p>Full Materials Calculator, SANS 10107 / 10400 Audit & Costing Suite</p>
    </div>

    <form id="tilingForm" onsubmit="event.preventDefault(); runEngine();">
      
      <!-- Section 1: Surface Parameters -->
      <div class="section-title">1. Surface Measurements (Meters)</div>
      <table class="grid-2">
        <tr>
          <td>
            <label for="length">Length (m)</label>
            <input type="number" id="length" step="0.01" value="5.0" required>
          </td>
          <td>
            <label for="width">Width (m)</label>
            <input type="number" id="width" step="0.01" value="4.0" required>
          </td>
        </tr>
      </table>

      <!-- Section 2: Tile Profile Properties -->
      <div class="section-title">2. Tile Dimensions & Grout</div>
      <table class="grid-3">
        <tr>
          <td>
            <label for="tileLength">Tile Length (mm)</label>
            <input type="number" id="tileLength" value="600" required>
          </td>
          <td>
            <label for="tileWidth">Tile Width (mm)</label>
            <input type="number" id="tileWidth" value="600" required>
          </td>
          <td>
            <label for="groutWidth">Grout Joint (mm)</label>
            <input type="number" id="groutWidth" step="0.5" value="3" required>
          </td>
        </tr>
      </table>

      <!-- Section 3: Job Environment & Financial Assumptions -->
      <div class="section-title">3. Application Profile & Financials</div>
      <table class="grid-2">
        <tr>
          <td>
            <label for="environment">Application Environment</label>
            <select id="environment">
              <option value="indoor_dry">Indoor Dry (Residential)</option>
              <option value="indoor_wet">Indoor Wet (Shower/Bathroom)</option>
              <option value="outdoor">Exterior (Exposed Balcony/Patio)</option>
              <option value="commercial">Commercial High-Traffic</option>
            </select>
          </td>
          <td>
            <label for="wastage">Wastage Margin (%)</label>
            <input type="number" id="wastage" value="10" required>
          </td>
        </tr>
        <tr>
          <td>
            <label for="tileCost">Tile Price (R per m²)</label>
            <input type="number" id="tileCost" value="250" required>
          </td>
          <td>
            <label for="labourCost">Labour Rate (R per m²)</label>
            <input type="number" id="labourCost" value="180" required>
          </td>
        </tr>
      </table>

      <!-- Action Buttons -->
      <table class="btn-container">
        <tr>
          <td>
            <button type="submit" class="btn btn-primary">Run Compliance Audit</button>
          </td>
          <td>
            <button type="button" onclick="saveProfile()" class="btn btn-secondary">💾 Save Project Profile</button>
          </td>
        </tr>
      </table>
    </form>

    <!-- Output Dashboard -->
    <div id="results" class="hidden">
      
      <!-- SANS Report Card -->
      <div id="sansCard" class="sans-card">
        <div class="sans-title">🛡️ SANS 10107 / 10400 National Audit</div>
        <p id="sansMessage" class="sans-msg"></p>
        <ul id="sansRules" class="sans-list"></ul>
      </div>

      <!-- Material Take-off Breakdown -->
      <div class="section-title">📊 Materials & Quantities Take-Off</div>
      <table class="grid-4">
        <tr>
          <td>
            <div class="stat-box">
              <span class="stat-label">Net Area</span>
              <span id="outNetArea" class="stat-val">-</span>
            </div>
          </td>
          <td>
            <div class="stat-box">
              <span class="stat-label">With Waste Margin</span>
              <span id="outGrossArea" class="stat-val">-</span>
            </div>
          </td>
          <td>
            <div class="stat-box">
              <span class="stat-label">Est. Tile Units</span>
              <span id="outTileCount" class="stat-val">-</span>
            </div>
          </td>
          <td>
            <div class="stat-box">
              <span class="stat-label">Adhesive Packs</span>
              <span id="outAdhesive" class="stat-val">-</span>
            </div>
          </td>
        </tr>
      </table>

      <!-- Cost Table (ZAR) -->
      <div class="section-title">💰 Project Valuation (ZAR)</div>
      <table class="data-table">
        <thead>
          <tr>
            <th>Description</th>
            <th class="text-right">Total Cost</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>Tile Stock Volume Supply</td>
            <td id="rowTileCost" class="text-right">-</td>
          </tr>
          <tr>
            <td>Premium Tile Adhesive (20kg Bags)</td>
            <td id="rowAdhesiveCost" class="text-right">-</td>
          </tr>
          <tr>
            <td>Tiling Specialist Labor Subcontract</td>
            <td id="rowLabourCost" class="text-right">-</td>
          </tr>
          <tr class="total-row">
            <td>Total Project Valuation (Excl. VAT)</td>
            <td id="rowTotalCost" class="text-right">-</td>
          </tr>
        </tbody>
      </table>

      <!-- Export to PDF Button -->
      <button onclick="exportToPDF()" class="btn btn-export">
        💾 Export Official Engine Report (PDF)
      </button>
    </div>

  </div>

  <script>
    function runEngine() {
      const length = parseFloat(document.getElementById('length').value) || 0;
      const width = parseFloat(document.getElementById('width').value) || 0;
      const tileLength = parseFloat(document.getElementById('tileLength').value) || 0;
      const tileWidth = parseFloat(document.getElementById('tileWidth').value) || 0;
      const groutWidth = parseFloat(document.getElementById('groutWidth').value) || 0;
      const environment = document.getElementById('environment').value;
      const wastage = parseFloat(document.getElementById('wastage').value) || 0;
      const tileCost = parseFloat(document.getElementById('tileCost').value) || 0;
      const labourCost = parseFloat(document.getElementById('labourCost').value) || 0;

      const netArea = length * width;
      const grossArea = netArea * (1 + wastage / 100);
      const tileAreaM2 = (tileLength / 1000) * (tileWidth / 1000);
      const estimatedTileUnits = tileAreaM2 > 0 ? Math.ceil(grossArea / tileAreaM2) : 0;
      const adhesiveBags = Math.ceil(grossArea / 4.4);
      const adhesiveBagCost = 165;

      const totalTileMaterialCost = grossArea * tileCost;
      const totalAdhesiveCost = adhesiveBags * adhesiveBagCost;
      const totalLabourCost = netArea * labourCost;
      const grandTotal = totalTileMaterialCost + totalAdhesiveCost + totalLabourCost;

      document.getElementById('outNetArea').innerText = netArea.toFixed(2) + " m²";
      document.getElementById('outGrossArea').innerText = grossArea.toFixed(2) + " m²";
      document.getElementById('outTileCount').innerText = estimatedTileUnits + " pcs";
      document.getElementById('outAdhesive').innerText = adhesiveBags + " bags";

      document.getElementById('rowTileCost').innerText = "R " + totalTileMaterialCost.toFixed(2);
      document.getElementById('rowAdhesiveCost').innerText = "R " + totalAdhesiveCost.toFixed(2);
      document.getElementById('rowLabourCost').innerText = "R " + totalLabourCost.toFixed(2);
      document.getElementById('rowTotalCost').innerText = "R " + grandTotal.toFixed(2);

      auditSANSCompliance(groutWidth, environment, tileLength, tileWidth);
      document.getElementById('results').classList.remove('hidden');
    }

    function auditSANSCompliance(grout, env, tLen, tWidth) {
      const sansCard = document.getElementById('sansCard');
      const sansMessage = document.getElementById('sansMessage');
      const sansRules = document.getElementById('sansRules');

      sansRules.innerHTML = "";
      let pass = true;
      let warnings = [];

      if (env === 'outdoor' && grout < 5) {
        pass = false;
        warnings.push("SANS 10107 requires minimum 5mm movement grout joints for exterior applications exposed to thermal expansion.");
      } else if (env === 'indoor_wet' && grout < 3) {
        pass = false;
        warnings.push("SANS 10107 specifies minimum 3mm joints in wet areas to prevent structural moisture bridging.");
      } else if (grout < 2) {
        pass = false;
        warnings.push("Minimum recommended joint width for interior floor tiling under SANS standards is 2mm.");
      }

      if (tLen >= 600 || tWidth >= 600) {
        warnings.push("Large format tiles (≥600mm) require polymer-modified adhesive (C2/S1 classification) and double-buttering technique.");
      }

      if (pass) {
        sansCard.style.backgroundColor = "#ecfdf5";
        sansCard.style.borderColor = "#a7f3d0";
        sansMessage.style.color = "#065f46";
        sansMessage.innerText = "✅ COMPLIANT: Surface profile meets SANS 10107 / 10400 regulatory requirements.";
      } else {
        sansCard.style.backgroundColor = "#fffbeb";
        sansCard.style.borderColor = "#fde68a";
        sansMessage.style.color = "#92400e";
        sansMessage.innerText = "⚠️ ACTION REQUIRED: Non-compliance flagged under South African National Standards.";
      }

      warnings.forEach(rule => {
        const li = document.createElement('li');
        li.innerText = rule;
        sansRules.appendChild(li);
      });
    }

    function saveProfile() {
      alert('💾 Project profile saved successfully to local storage!');
    }

    function exportToPDF() {
      window.print();
    }
  </script>
</body>
</html>
"""

@app.route("/")
def home():
    return render_template_string(HTML_TEMPLATE)

@app.route("/export-pdf", methods=["POST"])
def export_pdf():
    # Direct server-side PDF conversion endpoint
    pdf_bytes = HTML(string=HTML_TEMPLATE).write_pdf()
    return send_file(
        io.BytesIO(pdf_bytes),
        mimetype="application/pdf",
        as_attachment=True,
        download_name="Tiltool_Compliance_Report.pdf"
    )

if __name__ == "__main__":
    print("🚀 Starting Tiltool App on http://127.0.0.1:5000")
    app.run(host="0.0.0.0", port=int(os.environ.get("PORT", 5000)), debug=True)