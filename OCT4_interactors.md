---
layout: default
title: OCT4 Protein Interactions
description: Predicted by AF2 Multimer structures of Oct4 with nuclear proteins
---

# OCT4 Protein Interactions

Predicted by AF2 Multimer structures of Oct4 with nuclear proteins

[Back to Main Site](https://intbio.org/PTF_PPI/)

## Protein Visualization

<div class="text-center">
    <strong>OCT4</strong> + <strong id="partner-name">Protein Partner</strong>
</div>

<div id="viewport" style="width:100%; height:500px; border:1px solid #ddd; border-radius:8px; box-shadow:0 4px 6px rgba(0,0,0,0.1); margin-bottom:2rem; background-color:white;"></div>

## Protein Interaction Structures

| View | Gene | UniProt AC | IPTM (Predicted TM-score) | IPSAE (Predicted Aligned Error) | pDockQ (Predicted DockQ Score) | PDB |
|------|------|------------|---------------------------|---------------------------------|-------------------------------|-----|
{% for row in site.data.oct4_interactions %}
| <input type="checkbox" class="struct-toggle" id="struct-{{ forloop.index }}"> | {{ row.gene }} | {{ row.uniprot_ac }} | <span class="badge rounded-pill bg-primary" style="font-size:0.9em; padding:0.4em 0.6em;">{{ row.iptm }}</span> | <span class="badge rounded-pill bg-info text-dark" style="font-size:0.9em; padding:0.4em 0.6em;">{{ row.ipsae }}</span> | <span class="badge rounded-pill bg-success" style="font-size:0.9em; padding:0.4em 0.6em;">{{ row.pdockq }}</span> | [Download PDB]({{ row.pdb_file }}) |
{% endfor %}

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

    // Load structures
    window.structureComponents = [];
    {% for row in site.data.oct4_interactions %}
        window.stage.loadFile("{{ row.pdb_file }}").then(function(component) {
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
            window.structureComponents[{{ forloop.index0 }}] = component;
        });
    {% endfor %}

    // Add event listeners for checkboxes
    document.querySelectorAll('.struct-toggle').forEach((checkbox, index) => {
        checkbox.addEventListener('change', function() {
            if (window.structureComponents && window.structureComponents[index]) {
                window.structureComponents[index].setVisibility(this.checked);
                if (this.checked) {
                    window.structureComponents[index].autoView();
                    const proteinName = "{{ site.data.oct4_interactions[index].gene }}";
                    document.getElementById('partner-name').textContent = proteinName;
                }
            }
        });
    });
});
</script>

<style>
.oct4-label { color: #6b5b95; font-weight: bold; }
.partner-label { color: #d64161; font-weight: bold; }
.download-btn { 
    background-color: #6b5b95; 
    color: white; 
    border: none;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    text-decoration: none;
    font-size: 0.875rem;
}
.download-btn:hover { 
    background-color: #56497a; 
    color: white; 
}
</style>
