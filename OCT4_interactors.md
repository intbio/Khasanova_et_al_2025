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
document.addEventListener('DOMContentLoaded', function() {
  // Initialize NGL viewer
  window.stage = new NGL.Stage("viewport", { 
    backgroundColor: "white",
    clipNear: 0,
    clipFar: 100,
    clipDist: 10
  });
  window.stage.viewerControls.spin([0, 1, 0], 0.05);

  // Load data from CSV
  fetch('oct4_git.csv')
    .then(response => response.text())
    .then(data => {
      const rows = data.split('\n').slice(1);
      const table = $('#structures-table').DataTable({
        responsive: true,
        pageLength: 10
      });

      rows.forEach((row, index) => {
        if (!row.trim()) return;
        
        const [gene, uniprot_ac, iptm, ipsae, pdockq, pdbFile] = row.split(',');
        
        table.row.add([
          `<input type="checkbox" class="struct-toggle" id="struct-${index}">`,
          gene,
          uniprot_ac,
          `<span class="badge rounded-pill bg-primary badge-score">${iptm}</span>`,
          `<span class="badge rounded-pill bg-info text-dark badge-score">${ipsae}</span>`,
          `<span class="badge rounded-pill bg-success badge-score">${pdockq}</span>`,
          `<a href="structures/OCT4_high_quality/${pdbFile}" class="download-btn" download>Download</a>`
        ]).draw(false);
        
        // Load structure
        window.stage.loadFile(pdbFile).then(function(component) {
          component.setVisibility(false);
          component.addRepresentation('cartoon', {
            sele: ":A",
            color: "#6b5b95",
            aspectRatio: 2,
            radius: 1.5
          });
          component.addRepresentation('cartoon', {
            sele: ":B",
            color: "#d64161",
            aspectRatio: 2,
            radius: 1.5
          });
          
          if (!window.structureComponents) window.structureComponents = [];
          window.structureComponents[index] = component;
        });
      });

      // Toggle visibility
      $('#structures-table').on('change', '.struct-toggle', function() {
        const index = $(this).attr('id').split('-')[1];
        const isChecked = $(this).is(":checked");
        
        if (window.structureComponents && window.structureComponents[index]) {
          window.structureComponents[index].setVisibility(isChecked);
          if (isChecked) {
            window.structureComponents[index].autoView();
            const proteinName = $(this).closest('tr').find('td:eq(1)').text();
            $('#partner-name').text(proteinName);
          }
        }
      });
    })
    .catch(error => console.error('Error loading data:', error));
});
</script>
