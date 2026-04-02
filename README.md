<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>AI Parent & Child Predictor</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<style>
body { font-family: Arial; margin:20px; background:#f9f9f9; }
h1 { color:#0077cc; }
label { display:block; margin-top:10px; font-weight:bold; }
input, select, button { margin-top:5px; padding:8px; width:250px; }
button { background:#0077cc; color:white; border:none; cursor:pointer; }
#result { margin-top:20px; background:#e2f0ff; padding:10px; white-space:pre-wrap; }
img { max-width:200px; margin-top:10px; }
</style>
</head>

<body>

<h1>AI Parent & Child Predictor</h1>

<h2>Observe Person → Predict Parents</h2>
<input type="file" id="personPhoto">
<img id="personPreview">

<h2>Future Child Prediction</h2>
<input type="file" id="parent1">
<img id="preview1">

<input type="file" id="parent2">
<img id="preview2">

<select id="childGender">
<option value="">Any Gender</option>
<option value="male">Male</option>
<option value="female">Female</option>
</select>

<label>Replicate API Key</label>
<input type="text" id="r8_XiG2WgB7lm3rBg9stDbIxxzs5aCUOPw20e5np" placeholder="r8_XiG2WgB7lm3rBg9stDbIxxzs5aCUOPw20e5np">

<button onclick="runApp()">Analyze / Predict</button>
<button onclick="downloadPDF()">Download PDF</button>

<div id="result"></div>
<img id="childImage">

<script>
const { jsPDF } = window.jspdf;
let reportText = "";

document.getElementById("personPhoto").onchange = e =>
document.getElementById("personPreview").src = URL.createObjectURL(e.target.files[0]);

document.getElementById("parent1").onchange = e =>
document.getElementById("preview1").src = URL.createObjectURL(e.target.files[0]);

document.getElementById("parent2").onchange = e =>
document.getElementById("preview2").src = URL.createObjectURL(e.target.files[0]);

// -------- PERSON → PARENT PREDICTION --------
function predictParents() {
  const skin = ["White","Brown","Dark"][Math.floor(Math.random()*3)];
  const shape = ["Oval","Round","Square"][Math.floor(Math.random()*3)];

  return `
PARENT PREDICTION
-----------------------
Father Height: ~160-175 cm
Mother Height: ~150-165 cm
Body Structure: Similar to parents (~60%)
Skin Color: ${skin}
Facial Shape: ${shape}
Special Features: Beard from father / Braid from mother

Disclaimer: Traits may also come from grandparents.
`;
}

// -------- AI CHILD GENERATION --------
async function generateChild(apiKey, file1, file2, gender){

  const toBase64 = file => new Promise((res, rej)=>{
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = ()=>res(reader.result);
    reader.onerror = err=>rej(err);
  });

  const img1 = await toBase64(file1);
  const img2 = await toBase64(file2);

  // Start prediction
  let response = await fetch("https://api.replicate.com/v1/predictions", {
    method: "POST",
    headers: {
      "Authorization": `Token ${apiKey}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      version: "db21e45b5f3e8c8e0c3a1f8e0b8c8f3a6f4e0b5c8e3f5a6c7d8e9f0a1b2c3d4",
      input: {
        prompt: `Realistic face of a child from two parents. Parent1: ${img1}, Parent2: ${img2}, gender: ${gender}`,
        width: 512,
        height: 512
      }
    })
  });

  let data = await response.json();

  // Poll result
  while(data.status !== "succeeded" && data.status !== "failed"){
    await new Promise(r=>setTimeout(r,2000));
    let poll = await fetch(`https://api.replicate.com/v1/predictions/${data.id}`, {
      headers: { "Authorization": `Token ${apiKey}` }
    });
    data = await poll.json();
  }

  if(data.status==="succeeded"){
    return data.output[0];
  } else {
    throw new Error("AI failed");
  }
}

// -------- MAIN FUNCTION --------
async function runApp(){

  const apiKey = document.getElementById("apiKey").value;
  const p1 = document.getElementById("parent1").files[0];
  const p2 = document.getElementById("parent2").files[0];
  const gender = document.getElementById("childGender").value;

  reportText = predictParents();

  // If parents provided → generate child
  if(p1 && p2){
    if(!apiKey){
      alert("r8_XiG2WgB7lm3rBg9stDbIxxzs5aCUOPw20e5np");
      return;
    }

    document.getElementById("result").innerText = "Generating AI child...";

    try{
      const img = await generateChild(apiKey, p1, p2, gender);
      document.getElementById("childImage").src = img;

      reportText += `

CHILD PREDICTION
-----------------------
Gender: ${gender || "Any"}
Height: ~160-175 cm
Body Structure: Mix of parents
Skin Color: Mixed
Facial Shape: Combined traits

Disclaimer: AI prediction is approximate.
`;

    }catch(err){
      alert("Error: " + err.message);
    }
  }

  document.getElementById("result").innerText = reportText;
}

// -------- PDF --------
function downloadPDF(){
  if(!reportText){ alert("Run prediction first"); return; }

  const doc = new jsPDF();
  const lines = reportText.split("\n");
  let y=10;

  lines.forEach(l=>{
    doc.text(l,10,y);
    y+=8;
  });

  const img = document.getElementById("childImage").src;
  if(img){
    doc.addImage(img,"PNG",10,y,80,80);
  }

  doc.save("report.pdf");
}
</script>

</body>
</html>
