---
layout: default
title: SOX2 Protein Interactions
description: Predicted by AF2 Multimer structures of Sox2 with nuclear proteins
---

<style>
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background-color: #f8f9fa;
  }
  .header {
    background: linear-gradient(135deg, #00fffb 0%, #0ca9a7 100%);
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
  .sox2-label {
    color: #00fffb;
    font-weight: bold;
    text-shadow: 
    0 0 2px #0ca9a7,
    0 0 2px #0ca9a7,
    0 0 2px #0ca9a7,
    0 0 2px #0ca9a7;
  }
  .partner-label {
  color: #d3d3d3;
  font-weight: bold;
  text-shadow: 
    0 0 2px #000,
    0 0 2px #000,
    0 0 2px #000,
    0 0 2px #000;
}
  .badge-score {
    font-size: 0.9em;
    padding: 0.4em 0.6em;
  }
  .download-btn {
    background-color: #0ca9a7;
    color: white;
    border: none;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    text-decoration: none;
  }
  .download-btn:hover {
    background-color: #0ca9a7;
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
/* Стили для ссылок */
.gene-link, .uniprot-link {
  color: #6b5b95;
  text-decoration: none;
  font-weight: 500;
  transition: all 0.2s;
  display: inline-block;
  padding: 2px 0;
}

.gene-link:hover, .uniprot-link:hover {
  color: #d64161;
  text-decoration: underline;
}

/* Для DataTables */
#structures-table th.dt-center, 
#structures-table td.dt-center {
  text-align: center;
}
</style>

<div class="header text-center">
  <div class="container">
    <h1>SOX2 Protein Interactions</h1>
    <p>Predicted by AF2 Multimer structures of Oct4 with nuclear proteins</p>
    <a href="https://intbio.org/Khasanova_et_al_2025/" class="btn btn-light">Back to Main Site</a>
  </div>
</div>

<div class="container">
  <div class="protein-title text-center">
    <span class="sox2-label">SOX2</span> + 
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
  fetch('sox2_git.csv')
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
  data: dataRows.map((row, index) => {
    const cols = row.split(',');
    if (cols.length < 6) {
      console.warn("Invalid row:", row);
      return null;
    }
    
    const gene = cols[0].trim();
    const uniprot = cols[1].trim();
    const pdbFile = cols[5].trim();
    
    return {
      // Данные должны соответствовать названиям в columns.data
      view: `<input type="checkbox" class="struct-toggle" id="struct-${index}" data-pdb="${pdbFile}" data-gene="${gene}">`,
      gene: gene, // Простой текст для сортировки
      gene_link: `<a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=${gene}" target="_blank" class="gene-link">${gene}</a>`,
      uniprot: uniprot, // Простой текст для сортировки
      uniprot_link: `<a href="https://www.uniprot.org/uniprotkb/${uniprot}/entry" target="_blank" class="uniprot-link">${uniprot}</a>`,
      iptm: cols[2].trim(),
      ipsae: cols[3].trim(),
      pdockq: cols[4].trim(),
      pdb: `<a href="structures/SOX2_high_quality/${pdbFile}" class="download-btn" download>Download</a>`
    };
  }).filter(Boolean),
  columns: [
    { 
      data: 'view',
      title: "View",
      orderable: false,
      className: "dt-center"
    },
    { 
      data: 'gene_link',
      title: "Gene",
      render: function(data, type) {
        // Для сортировки/фильтрации используем plain text
        if (type === 'sort' || type === 'filter') {
          return $(data).text() || data;
        }
        return data;
      }
    },
    { 
      data: 'uniprot_link',
      title: "UniProt AC",
      render: function(data, type) {
        if (type === 'sort' || type === 'filter') {
          return $(data).text() || data;
        }
        return data;
      }
    },
    { 
      data: 'iptm',
      title: "ipTM",
      render: function(data) {
        return `<span class="badge rounded-pill bg-primary">${data}</span>`;
      },
      className: "dt-center"
    },
    { 
      data: 'ipsae',
      title: "ipSAE",
      render: function(data) {
        return `<span class="badge rounded-pill bg-info text-dark">${data}</span>`;
      },
      className: "dt-center"
    },
    { 
      data: 'pdockq',
      title: "pDockQ",
      render: function(data) {
        return `<span class="badge rounded-pill bg-success">${data}</span>`;
      },
      className: "dt-center"
    },
    { 
      data: 'pdb',
      title: "PDB",
      orderable: false,
      className: "dt-center"
    }
  ],
  createdRow: function(row, data) {
    // Добавляем data-атрибуты для строки
    $(row).attr('data-gene', data.gene);
    $(row).attr('data-uniprot', data.uniprot);
  }
});

      // Обработчик чекбоксов с улучшенной загрузкой
      // Глобальные переменные
window.selectedProteins = []; // Хранит названия выбранных белков
window.structureComponents = {}; // Хранит загруженные компоненты
window.loadedStructures = []; // Хранит загруженные структуры для выравнивания

// Обработчик чекбоксов
$('#structures-table').on('change', '.struct-toggle', async function() {
  const pdbFile = $(this).data('pdb');
  const geneName = $(this).data('gene');
  const isChecked = $(this).is(":checked");
  
  try {
    if (isChecked) {
      // Добавляем белок в список выбранных (если ещё не добавлен)
      if (!window.selectedProteins.some(p => p.name === geneName)) {
        window.selectedProteins.push({ name: geneName, file: pdbFile });
      }

      if (!window.structureComponents[pdbFile]) {
        console.log(`Loading structure: ${pdbFile}`);
        
        // Пробуем разные пути загрузки
        const pathsToTry = [
          `structures/SOX2_high_quality/${pdbFile}`,
          `structures/SOX2_high_quality/${pdbFile}.pdb`,
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
        
        // Представления структуры
        component.addRepresentation('cartoon', {
          sele: ":A", color: "#ffa533", aspectRatio: 2, radius: 1.5
        });
        component.addRepresentation('cartoon', {
          sele: ":B", color: "#d3d3d3", aspectRatio: 2, radius: 1.5 
        });
        
        // Выделение ДНК-связывающего домена
        const dnaSel = ":A and (143-212 or 231-287)";
        component.autoView(dnaSel, 1000);
        
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
      } else {
        window.structureComponents[pdbFile].setVisibility(true);
      }
    } else {
      // Удаляем белок из списка выбранных
      window.selectedProteins = window.selectedProteins.filter(p => p.name !== geneName);
      window.structureComponents[pdbFile]?.setVisibility(false);
    }
    
    // Обновляем заголовок
    updateProteinTitle();
    
  } catch (error) {
    console.error("Structure loading failed:", error);
    $(this).prop('checked', false);
    alert(`Failed to load structure:\n${pdbFile}\nError: ${error.message}`);
  }
});
      // Функция обновления заголовка
// Функция обновления заголовка
function updateProteinTitle() {
  if (window.selectedProteins.length === 0) {
    // Если ничего не выбрано, показываем стандартный текст
    $('#partner-name').html(`<span class="partner-label">Protein Partner</span>`);
  } else {
    // Формируем список выбранных партнеров
    const partnerLabels = window.selectedProteins.map(protein => 
      `<span class="partner-label" data-pdb="${protein.file}">${protein.name}</span>`
    ).join(' + ');
    
    // Обновляем заголовок (без повторного SOX2)
    $('#partner-name').html(partnerLabels);
  }
}
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
