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
    background: linear-gradient(135deg, #ffa533 0%, #a188d6 100%);
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
    color: #ffa533;
    font-weight: bold;
  }
  .partner-label {
    color: #d3d3d3;
    font-weight: bold;
  }
  .badge-score {
    font-size: 0.9em;
    padding: 0.4em 0.6em;
  }
  .download-btn {
    background-color: #ffa533;
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
  #structures-table thead th.sorting { 
  cursor: pointer;
  position: relative;
  padding-right: 20px;
}
  #structures-table thead th.sorting:after {
  content: "⇅";
  position: absolute;
  right: 5px;
  top: 50%;
  transform: translateY(-50%);
  opacity: 0.3;
}
  #structures-table thead th.sorting_asc:after {
  content: "↑";
  opacity: 1;
}
  #structures-table thead th.sorting_desc:after {
  content: "↓";
  opacity: 1;
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
  // Инициализация
  window.stage = new NGL.Stage("viewport", {
    backgroundColor: "white",
    clipNear: 0,
    clipFar: 100,
    clipDist: 10
  });
  window.structureComponents = {};
  window.loadedStructures = [];
  
  // Загрузка данных
  fetch('oct4_git.csv')
    .then(response => {
      if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
      return response.text();
    })
    .then(csvData => {
      const rows = csvData.split('\n').filter(row => row.trim());
      if (rows.length < 2) throw new Error("CSV file is empty or has no data rows");
      
      const headers = rows[0].split(',');
      const dataRows = rows.slice(1);

      // Инициализация таблицы
      const table = $('#structures-table').DataTable({
        data: dataRows.map(row => {
          const cols = row.split(',');
          if (cols.length < 6) {
            console.warn("Invalid row:", row);
            return null;
          }
          const pdbFile = cols[5].trim();
          return [
            `<input type="checkbox" class="struct-toggle" data-pdb="${pdbFile}" data-gene="${cols[0].trim()}">`,
            cols[0].trim(), // gene
            cols[1].trim(), // uniprot
            `<span class="badge rounded-pill bg-primary">${cols[2].trim()}</span>`,
            `<span class="badge rounded-pill bg-info text-dark">${cols[3].trim()}</span>`,
            `<span class="badge rounded-pill bg-success">${cols[4].trim()}</span>`,
            `<a href="structures/OCT4_high_quality/${pdbFile}" class="download-btn" download>Download</a>`
          ];
        }).filter(Boolean),
        columns: [
          { title: "View", orderable: false },
          { title: "Gene" },
          { title: "UniProt AC" },
          { title: "ipTM" },
          { title: "ipSAE" },
          { title: "pDockQ" },
          { title: "PDB", orderable: false }
        ]
      });

      // Обработчик чекбоксов с улучшенной загрузкой
      $('#structures-table').on('change', '.struct-toggle', async function() {
        const pdbFile = $(this).data('pdb');
        const geneName = $(this).data('gene');
        const isChecked = $(this).is(":checked");
        
        try {
          if (isChecked) {
            if (!window.structureComponents[pdbFile]) {
              console.log(`Loading structure: ${pdbFile}`);
              
              // Пробуем разные пути
              const pathsToTry = [
                `structures/OCT4_high_quality/${pdbFile}`,
                `structures/OCT4_high_quality/${pdbFile}.pdb`,
                pdbFile,
                `${pdbFile}.pdb`
              ];
              
              let component;
              for (const path of pathsToTry) {
                try {
                  console.log(`Trying path: ${path}`);
                  component = await window.stage.loadFile(path);
                  break;
                } catch (e) {
                  console.warn(`Failed to load from ${path}:`, e.message);
                }
              }
              
              if (!component) throw new Error(`All loading attempts failed for ${pdbFile}`);
              
              window.structureComponents[pdbFile] = component;
              window.loadedStructures.push(component.structure);
              
              // Добавляем представления
              component.addRepresentation('cartoon', {
                sele: ":A", color: "#ffa533", aspectRatio: 2, radius: 1.5
              });
              component.addRepresentation('cartoon', {
                sele: ":B", color: "#d3d3d3", aspectRatio: 2, radius: 1.5
              });
              
              // Выделяем ДНК-связывающий домен
              const dnaSel = ":A and (143-212 or 231-287)";
              component.addRepresentation('ball+stick', {
                sele: dnaSel, color: 'yellow', radius: 0.3
              });
              
              // Выравнивание если есть другие структуры
              if (window.loadedStructures.length > 1) {
                try {
                  await NGL.superpose(
                    window.loadedStructures,
                    `${dnaSel} and name CA`
                  );
                  console.log("Structures aligned successfully");
                } catch (superposeError) {
                  console.warn("Superposition failed:", superposeError);
                }
              }
              
              component.autoView(dnaSel, 1000);
            } else {
              window.structureComponents[pdbFile].setVisibility(true);
            }
            $('#partner-name').text(geneName);
          } else {
            window.structureComponents[pdbFile]?.setVisibility(false);
          }
        } catch (error) {
          console.error("Structure loading failed:", error);
          $(this).prop('checked', false);
          alert(`Failed to load structure:\n${pdbFile}\nError: ${error.message}`);
        }
      });
    })
    .catch(error => {
      console.error("Initialization failed:", error);
      $('#viewport').html(`
        <div class="alert alert-danger">
          <h4>Error</h4>
          <p>${error.message}</p>
          <p>Check console for details</p>
        </div>
      `);
    });
});
</script>
