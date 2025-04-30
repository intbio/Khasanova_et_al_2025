---
layout: default
title: OCT4 Protein Interactions
description: Predicted by AF2 Multimer structures of Oct4 with nuclear proteins
---

<style>
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background-color: #f8f9fa;
  }
  .header {
    background: linear-gradient(135deg, #6b5b95 0%, #a188d6 100%);
    color: white;
    padding: 2rem 0;
    margin-bottom: 2rem;
    border-radius: 0 0 10px 10px;
  }
  #viewport {
    width: 100%;
    height: 500px;
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    margin-bottom: 2rem;
    background-color: white;
  }
  .protein-title {
    font-size: 1.5rem;
    font-weight: 600;
    margin-bottom: 1rem;
  }
  .oct4-label {
    color: #6b5b95;
    font-weight: bold;
  }
  .partner-label {
    color: #d64161;
    font-weight: bold;
  }
  .badge-score {
    font-size: 0.9em;
    padding: 0.4em 0.6em;
  }
  .download-btn {
    background-color: #6b5b95;
    color: white;
    border: none;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    text-decoration: none;
  }
  .download-btn:hover {
    background-color: #56497a;
    color: white;
  }
</style>

<div class="header text-center">
  <div class="container">
    <h1>OCT4 Protein Interactions</h1>
    <p>Predicted by AF2 Multimer structures of Oct4 with nuclear proteins</p>
    <a href="https://intbio.org/PTF_PPI/" class="btn btn-light">Back to Main Site</a>
  </div>
</div>

<div class="container">
  <div class="protein-title text-center">
    <span class="oct4-label">OCT4</span> + 
    <span class="partner-label" id="partner-name">Protein Partner</span>
  </div>
  <div id="viewport"></div>

  <h2>Protein Interaction Structures</h2>
  <div class="table-responsive">
    <table id="structures-table" class="table table-hover table-striped">
      <thead>
        <tr>
          <th>View</th>
          <th>Gene</th>
          <th>UniProt AC</th>
          <th>ipTM</th>
          <th>ipSAE</th>
          <th>pDockQ</th>
          <th>PDB</th>
        </tr>
      </thead>
      <tbody>
        <!-- Data will be loaded dynamically -->
      </tbody>
    </table>
  </div>
</div>

<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
<script src="https://cdn.datatables.net/1.11.5/js/jquery.dataTables.min.js"></script>
<script src="https://unpkg.com/ngl@2.0.0-dev.35/dist/ngl.js"></script>

<script>
document.addEventListener('DOMContentLoaded', async function() {
  try {
    window.stage = new NGL.Stage("viewport", { 
      backgroundColor: "white",
      clipNear: 0,
      clipFar: 100,
      clipDist: 10
    });
    window.stage.viewerControls.spin([0, 1, 0], 0.05);
    window.structureComponents = {};

    console.log("Starting CSV data loading...");
    const response = await fetch('oct4_git.csv');
    if (!response.ok) throw new Error(`Failed to load CSV: ${response.status}`);
    const csvData = await response.text();
    
    const rows = csvData.split('\n').filter(row => row.trim());
    if (rows.length < 2) throw new Error("CSV file is empty or has no data");
    const headers = rows[0].split(',');
    const dataRows = rows.slice(1);

    console.log("Initializing DataTable...");
    const table = $('#structures-table').DataTable({
      responsive: true,
      pageLength: 10,
      data: dataRows.map((row, index) => {
        const cols = row.split(',');
        if (cols.length < 6) {
          console.warn(`Invalid row ${index}: ${row}`);
          return null;
        }
        
        const gene = cols[0].trim();
        const uniprot_ac = cols[1].trim();
        const iptm = cols[2].trim();
        const ipsae = cols[3].trim();
        const pdockq = cols[4].trim();
        let pdbFile = cols[5].trim();
        

        if (!pdbFile.toLowerCase().endsWith('.pdb')) {
          pdbFile += '.pdb';
        }
        
        return [
          `<input type="checkbox" class="struct-toggle" 
           data-pdb="${pdbFile}" data-gene="${gene}">`,
          gene,
          uniprot_ac,
          `<span class="badge rounded-pill bg-primary">${iptm}</span>`,
          `<span class="badge rounded-pill bg-info text-dark">${ipsae}</span>`,
          `<span class="badge rounded-pill bg-success">${pdockq}</span>`,
          `<a href="structures/OCT4_high_quality/${pdbFile}" class="download-btn" download>Download</a>`
        ];
      }).filter(row => row !== null),
      columns: [
        { title: "View", className: "dt-center" },
        { title: "Gene" },
        { title: "UniProt AC" },
        { title: "ipTM", className: "dt-center" },
        { title: "ipSAE", className: "dt-center" },
        { title: "pDockQ", className: "dt-center" },
        { title: "PDB", className: "dt-center" }
      ],
      createdRow: function(row, data, dataIndex) {
        $(row).attr('data-index', dataIndex);
      }
    });

    $('#structures-table').on('change', '.struct-toggle', async function() {
      const pdbFile = $(this).data('pdb');
      const geneName = $(this).data('gene');
      const isChecked = $(this).is(":checked");
      
      console.log(`Toggling ${pdbFile}, checked: ${isChecked}`);

      try {
        if (isChecked) {
          if (!window.structureComponents[pdbFile]) {
            console.log(`Loading structure: ${pdbFile}`);
            
            const filePath = `structures/OCT4_high_quality/${pdbFile}`;
            console.log(`Loading from path: ${filePath}`);
            
            const component = await window.stage.loadFile(filePath, { ext: "pdb" })
              .catch(async error => {
                console.warn(`First load attempt failed, trying alternative...`, error);
                return await window.stage.loadFile(filePath);
              });
            
            window.structureComponents[pdbFile] = component;
            
            try {
              component.addRepresentation('cartoon', {
                sele: ":A", color: "#6b5b95", aspectRatio: 2, radius: 1.5
              });
              component.addRepresentation('cartoon', {
                sele: ":B", color: "#d64161", aspectRatio: 2, radius: 1.5
              });
            } catch (repError) {
              console.warn("Error adding representations:", repError);
            }
          }
          
          window.structureComponents[pdbFile].setVisibility(true);
          window.structureComponents[pdbFile].autoView(500);
          $('#partner-name').text(geneName);
        } else {
          if (window.structureComponents[pdbFile]) {
            window.structureComponents[pdbFile].setVisibility(false);
          }
        }
      } catch (error) {
        console.error(`Error processing ${pdbFile}:`, error);
        $(this).prop('checked', false);
        
        alert(`Failed to load structure ${pdbFile}:\n${error.message}\n\nCheck console for details.`);
      }
    });

    console.log("Initialization completed successfully");
  } catch (error) {
    console.error("Initialization failed:", error);
    
    $('#viewport').html(`
      <div class="alert alert-danger p-3">
        <h4>Initialization Error</h4>
        <p>${error.message}</p>
        <p class="mb-0">Please check browser console (F12) for details.</p>
      </div>
    `);
  }
});
</script>
