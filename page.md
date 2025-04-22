## Предсказанные структуры комплексов ПТФ с белками хроматина
[Back](https://intbio.org/PTF_PPI/)

<html lang="en">
<head>
  <meta charset="utf-8">
</head>
<body>
<br>
  <p style="color:#d6d6d6;font-size:22px;font-family:verdana;font-weight: bold;text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;display: inline">Integrin aV (PDB ID 1L5G, chain A)</p>
  <br />
  <p style="color:#FFF2CC;font-size:22px;font-family:verdana;font-weight: bold;text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;display: inline">Integrin b3 (PDB ID 1L5G, chain B)</p>

<table border="solid 1px;" style="font-size:14px;">
<tr>
<th> Show </th> <th> ПТФ </th> <th> Белок-партнер, UniProt ID </th><th> Белок-партнер, ген </th><th> iPTM </th><th>ipSAE </th><th> pDockQ </th><th> Скачать структуру </th>
</tr>

<tbody>
  
  <script src="https://unpkg.com/ngl@2.0.0-dev.35/dist/ngl.js"></script>
  <script src="https://code.jquery.com/jquery-3.5.1.min.js" integrity="sha256-9/aliU8dGd2tb6OSsuzixeV4y/faTqgFtohetphbbj0=" crossorigin="anonymous"></script>
  <script>
  

   var names = ['structures/Q01860_Q9H867_aligned1.pdb',
 'structures/Q01860_P60510_aligned1.pdb',
 'structures/Q01860_Q96G04_aligned1.pdb',
 'structures/Q01860_P62249_aligned1.pdb']
   var models = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
   var genes = [0.676,0.676,0.676,0.636,]
   var proteins = [0.505,-7.235,-1.036,-4.986,]
   var iptm = ['NA','-12.3 ± 0.98','-10.1 ± 1.41','-12.78 ± 1.86','-5.5 ± 1.32',]
   var ipsae = ['NA','-12.3 ± 0.98','-10.1 ± 1.41','-12.78 ± 1.86','-5.5 ± 1.32',]
   var pdockq = ['NA','-12.3 ± 0.98','-10.1 ± 1.41','-12.78 ± 1.86','-5.5 ± 1.32',]
   peptide_reps = [];
    $(document).ready(function() {
      window.stage = new NGL.Stage("viewport",{ backgroundColor:"#FFFFFF" });
      window.stage.loadFile("structures/Q01860_P62249_aligned1.pdb").then(function (ref_pdb) {
        var aspectRatio = 2;
        var radius = 1.5;

        ref_pdb.addRepresentation('cartoon', {
           "sele": ":A", "color": 0xd6d6d6,"aspectRatio":aspectRatio, "radius":radius,"radiusSegments":1,"capped":0 });;
	ref_pdb.addRepresentation('cartoon', {
           "sele": ":B", "color": '#FFF2CC',"aspectRatio":aspectRatio, "radius":radius,"radiusSegments":1,"capped":0 });;
        ref_pdb.autoView();
      });

      var arrayLength = names.length;
      var k;

  //   var hyper_scheme = NGL.ColormakerRegistry.addSelectionScheme([
  //       ["orange", ".CA"],
  //       ['0xecf0f1', '_H'],
  //       ["blue", "_N"],
  //       ["red", "_O"],
  //       ["cyan", "*"]
  //     ], "DA");
		// for (k = 0; k < arrayLength; k++) {
  //           window.stage.loadFile(`${names[k]}`).then(function (ref_pdb) {
  //               var repr = ref_pdb.addRepresentation('hyperball', {
  //                  "sele": ":C", "color": hyper_scheme});
  //               repr.setVisibility(false);
  //               peptide_reps.push(repr);
               
  //         	});
		// }

    window.stage.viewerControls.spin( [ 0, 1, 0 ],110 )
    });
    var arrayLength = names.length;
			for (var i = 0; i < arrayLength; i++) {
        
        document.write(`<tr><td> <input type="checkbox" id="${i}" name="${genes[i]}"></td> <td>  ${proteins[i]}  </td> <td> ${iptm[i]} </td><td> ${ipsae[i]} </td></td><td> ${pdockq[i]} </td><td> <a href="https://intbio.org/PTF_PPI/${names[i]}" download>PDB</a> </td></tr>`); 
			}
		  
      
$('input[type=checkbox]').on('change', toggle_reference_structure);

function toggle_reference_structure() {
               var state = $(this).is(":checked");
               var nameid = $(this).attr('id');
               peptide_reps[nameid].setVisibility(state)
          }

  </script>
  <div id="viewport" style="width:500px; height:500px; border: thin solid black"></div>
  </tbody>	
</table>
</body>
</html>
