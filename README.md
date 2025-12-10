<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>LoanPro App Final</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
:root{
  --main:#0047b3;
  --bg:#e6f7ff;
  --text:18px;
}

body{
  background:var(--bg);
  font-family:Arial;
  margin:0;
  padding:10px;
  font-size:var(--text);
}

.box{
  max-width:420px;
  margin:auto;
  background:white;
  padding:15px;
  border-radius:14px;
  box-shadow:0 5px 15px rgba(0,0,0,0.2);
  margin-bottom:12px;
}

h2{text-align:center;color:var(--main)}

input,button,select{
  width:100%;
  padding:12px;
  margin-top:10px;
  font-size:var(--text);
  border-radius:10px;
  border:2px solid #444;
}

button{
  background:var(--main);
  color:white;
  border:none;
}

#result,#grand{
  background:black;
  color:#0f0;
  padding:10px;
  margin-top:12px;
  border-radius:10px;
  text-align:center;
}

table{width:100%;border-collapse:collapse;margin-top:12px}
th,td{border:1px solid #777;padding:8px;text-align:center}
th{background:var(--main);color:white}
</style>
</head>

<body>

<div class="box">
<h2>LoanPro Mobile App</h2>

<input id="billNo" placeholder="Enter BILL NO" autofocus onkeydown="jumpFetch(event)">
<button onclick="fetchBill()">FETCH</button>

<input id="amount" type="number" placeholder="Amount" onkeydown="jumpDate(event)">
<input id="date" type="date" onkeydown="jumpCalc(event)">
<button onclick="calculate()">CALCULATE</button>

<div id="result"></div>
<button onclick="save()">➕ ADD ENTRY</button>
</div>

<div class="box">
<table>
<tr><th>Amount</th><th>Months</th><th>Total ₹</th></tr>
<tbody id="data"></tbody>
</table>
<div id="grand">Grand Total: ₹0</div>
</div>

<!-- SETTINGS BOX -->
<div class="box">
<button onclick="askAdmin()">⚙ SETTINGS</button>

<div id="settings" style="display:none">

<label>Main Color</label>
<input type="color" onchange="setColor(this.value)">

<label>Dark / Light Mode</label>
<select onchange="setMode(this.value)">
<option value="light">Light</option>
<option value="dark">Dark</option>
</select>

<label>Font Size</label>
<select onchange="setFont(this.value)">
<option value="16px">Small</option>
<option value="18px" selected>Medium</option>
<option value="22px">Large</option>
</select>

<hr>

<label>Google Sheet URL</label>
<input id="sheetURL" placeholder="Paste Google Sheet API URL" onchange="saveSheetURL(this.value)">

<label>Change Admin Password</label>
<input id="newPass" placeholder="New Password" onchange="changePass(this.value)">

<button onclick="window.print()">🖨 PRINT</button>
</div>
</div>

<script>
let final={}, totalSum=0;
let ADMIN_PASS = localStorage.getItem("adminPass") || "1234";

/************ LOAD SETTINGS *************/
window.onload = function(){
  let u=localStorage.getItem("sheetURL");
  if(u) sheetURL.value=u;

  let f=localStorage.getItem("fontsize");
  if(f) document.documentElement.style.setProperty("--text",f);

  let c=localStorage.getItem("maincolor");
  if(c) document.documentElement.style.setProperty("--main",c);
};

/************ AUTO JUMP *************/
function jumpFetch(e){ if(e.key==="Enter"){ e.preventDefault(); fetchBill(); } }
function jumpDate(e){ if(e.key==="Enter"){ e.preventDefault(); date.focus(); } }
function jumpCalc(e){ if(e.key==="Enter"){ e.preventDefault(); calculate(); } }

/************ CALCULATION *************/
function calculate(){
 let amt=parseFloat(amount.value);
 let dt=date.value;
 if(!amt||!dt) return alert("Enter Amount & Date");

 let d1=new Date(dt), d2=new Date();
 let months=Math.floor((d2-d1)/(1000*60*60*24*30));
 if(months<1) months=1;

 let interest=0;
 if(months<=6){
  interest=months*(amt*0.0125);
 }else{
  interest=6*(amt*0.0125);
  interest+=10;
  interest+=(months-6)*(amt*0.0175);
 }

 let total=amt+interest;
 final={amt,months,total};

 result.innerHTML="Months: "+months+
 "<br>Interest: ₹"+interest.toFixed(2)+
 "<br><b>Total: ₹"+total.toFixed(2)+"</b>";
}

/************ SAVE ENTRY *************/
function save(){
 if(!final.total) return alert("Calculate First");
 let tr=document.createElement("tr");
 tr.innerHTML="<td>"+final.amt+"</td><td>"+final.months+"</td><td>₹"+final.total.toFixed(2)+"</td>";
 data.appendChild(tr);
 totalSum+=final.total;
 grand.innerText="Grand Total: ₹"+totalSum.toFixed(2);
}

/************ ✅ GOOGLE SHEET FETCH (FINAL) *************/
async function fetchBill(){

 let bill=billNo.value.trim();
 if(!bill) return alert("Enter BILL NO");

 let baseURL=localStorage.getItem("sheetURL");
 if(!baseURL) return alert("Set Google Sheet URL in Settings");

 let url=baseURL+"?bill="+encodeURIComponent(bill);

 try{
  let res=await fetch(url);
  let data=await res.json();

  if(data.error){
   alert("Bill Not Found");
   return;
  }

  amount.value=data.amount;
  date.value=formatDate(data.date);
  calculate();

 }catch(e){
  alert("Google Sheet Connection Error");
 }
}

function formatDate(d){
 let dt=new Date(d);
 let m=("0"+(dt.getMonth()+1)).slice(-2);
 let day=("0"+dt.getDate()).slice(-2);
 return dt.getFullYear()+"-"+m+"-"+day;
}

/************ SETTINGS *************/
function askAdmin(){
 let p=prompt("Enter Admin Password");
 if(p===ADMIN_PASS){
  settings.style.display=(settings.style.display=="none")?"block":"none";
 }else alert("Wrong Password");
}

function setColor(c){
 localStorage.setItem("maincolor",c);
 document.documentElement.style.setProperty("--main",c);
}

function setMode(m){
 if(m==="dark"){
  document.body.style.background="#111";
  document.body.style.color="white";
 }else{
  document.body.style.background="var(--bg)";
  document.body.style.color="black";
 }
}

function setFont(f){
 localStorage.setItem("fontsize",f);
 document.documentElement.style.setProperty("--text",f);
}

function saveSheetURL(url){
 localStorage.setItem("sheetURL",url);
 alert("Google Sheet URL Saved");
}

function changePass(p){
 localStorage.setItem("adminPass",p);
 ADMIN_PASS=p;
 alert("Password Changed");
}
</script>

</body>
</html>
