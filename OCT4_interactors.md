<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OCT4 Protein Interactions</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.datatables.net/1.11.5/css/dataTables.bootstrap5.min.css">
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
        .card {
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            margin-bottom: 2rem;
            border: none;
        }
        .card-header {
            background-color: #6b5b95;
            color: white;
            font-weight: 600;
            border-radius: 10px 10px 0 0 !important;
        }
        .table-responsive {
            border-radius: 0 0 10px 10px;
        }
        .badge-score {
            font-size: 0.9em;
            padding: 0.4em 0.6em;
        }
        .download-btn {
            background-color: #6b5b95;
            color: white;
            border: none;
        }
        .download-btn:hover {
            background-color: #56497a;
            color: white;
        }
    </style>
</head>
<body>
    <div class="header text-center">
        <div class="container">
            <h1 class="display-4">OCT4 Protein Interactions</h1>
            <p class="lead">Predicted by AF2 Multimer structures of Oct4 with nuclear proteins</p>
            <a href="https://intbio.org/PTF_PPI/" class="btn btn-light mt-2">Back to Main Site</a>
        </div>
    </div>

    <div class="container">
        <div class="row mb-4">
            <div class="col-md-12">
                <div class="protein-title text-center">
                    <span class="oct4-label">OCT4</span> + 
                    <span class="partner-label" id="partner-name">Protein Partner</span>
                </div>
                <div id="viewport"></div>
            </div>
        </div>

        <div class="row">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="mb-0">Protein Interaction Structures</h3>
                    </div>
                    <div class="card-body p-0">
                        <div class="table-responsive">
                            <table id="structures-table" class="table table-hover table-striped" style="width:100%">
                                <thead>
                                    <tr>
                                        <th>View</th>
                                        <th>Gene</th>
                                        <th>UniProt AC</th>
                                        <th>IPTM <i class="bi bi-info-circle" data-bs-toggle="tooltip" title="Predicted TM-score"></i></th>
                                        <th>IPSAE <i class="bi bi-info-circle" data-bs-toggle="tooltip" title="Predicted Aligned Error"></i></th>
                                        <th>pDockQ <i class="bi bi-info-circle" data-bs-toggle="tooltip" title="Predicted DockQ Score"></i></th>
                                        <th>PDB</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <!-- Data will be loaded from CSV -->
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Scripts -->
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    <script src="https://cdn.datatables.net/1.11.5/js/jquery.dataTables.min.js"></script>
    <script src="https://cdn.datatables.net/1.11.5/js/dataTables.bootstrap5.min.js"></script>
    <script src="https://unpkg.com/ngl@2.0.0-dev.35/dist/ngl.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <script>
        // Initialize NGL Stage
        document.addEventListener('DOMContentLoaded', function() {
            // Initialize tooltips
            const tooltipTriggerList = [].slice.call(document.querySelectorAll('[data-bs-toggle="tooltip"]'));
            tooltipTriggerList.map(function (tooltipTriggerEl) {
                return new bootstrap.Tooltip(tooltipTriggerEl);
            });

            // Initialize NGL viewer
            window.stage = new NGL.Stage("viewport", { 
                backgroundColor: "white",
                clipNear: 0,
                clipFar: 100,
                clipDist: 10
            });
            window.stage.viewerControls.spin([0, 1, 0], 0.05);

            // Load data from CSV
            fetch('./oct4_git.csv')
                .then(response => response.text())
                .then(data => {
                    const rows = data.split('\n').slice(1);
                    const tableBody = $('#structures-table tbody');
                    
                    // Initialize DataTable
                    const table = $('#structures-table').DataTable({
                        responsive: true,
                        columns: [
                            { data: 'view', orderable: false },
                            { data: 'gene' },
                            { data: 'uniprot ac' },
                            { data: 'iptm' },
                            { data: 'ipsae' },
                            { data: 'pdockq' },
                            { data: 'pdb', orderable: false }
                        ],
                        pageLength: 10,
                        lengthMenu: [5, 10, 25, 50]
                    });

                    // Process each row
                    rows.forEach((row, index) => {
                        if (!row.trim()) return;
                        
                        const [gene, uniprot_ac, iptm, ipsae, pdockq, pdbFile] = row.split(',');
                        
                        // Add to DataTable
                        table.row.add({
                            'view': `<input type="checkbox" class="struct-toggle" id="struct-${index}">`,
                            'gene': gene,
                            'uniprot ac': protein,
                            'iptm': `<span class="badge rounded-pill bg-primary badge-score">${iptm}</span>`,
                            'ipsae': `<span class="badge rounded-pill bg-info text-dark badge-score">${ipsae}</span>`,
                            'pdockq': `<span class="badge rounded-pill bg-success badge-score">${pdockq}</span>`,
                            'pdb': `<a href="${pdbFile}" class="btn btn-sm download-btn" download>Download PDB</a>`
                        }).draw(false);
                        
                        // Load structure
                        window.stage.loadFile(pdbFile).then(function(component) {
                            component.setVisibility(false);
                            
                            // Add representations
                            component.addRepresentation('cartoon', {
                                sele: ":A",
                                color: "#6b5b95",  // OCT4 color
                                aspectRatio: 2,
                                radius: 1.5
                            });
                            
                            component.addRepresentation('cartoon', {
                                sele: ":B",
                                color: "#d64161",  // Partner color
                                aspectRatio: 2,
                                radius: 1.5
                            });
                            
                            // Store component reference
                            if (!window.structureComponents) window.structureComponents = [];
                            window.structureComponents[index] = component;
                        });
                    });

                    // Add event listener for checkboxes
                    $('#structures-table').on('change', '.struct-toggle', function() {
                        const index = $(this).attr('id').split('-')[1];
                        const isChecked = $(this).is(":checked");
                        
                        if (window.structureComponents && window.structureComponents[index]) {
                            window.structureComponents[index].setVisibility(isChecked);
                            
                            if (isChecked) {
                                window.structureComponents[index].autoView();
                                // Update partner name
                                const proteinName = table.row($(this).closest('tr')).data().protein;
                                $('#partner-name').text(proteinName);
                            }
                        }
                    });
                })
                .catch(error => console.error('Error loading data:', error));
        });
    </script>
</body>
</html>
