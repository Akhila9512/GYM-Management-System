#GYM Management System 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gym Management System</title>
<style>
body{margin:0;font-family:Arial,sans-serif;background:#111827;color:#fff}
.login,.main{max-width:1200px;margin:auto;padding:20px}
.card{background:#1f2937;padding:20px;border-radius:12px;margin:10px 0}
input,button,select{padding:10px;margin:5px;border-radius:8px;border:none}
button{background:#2563eb;color:#fff;cursor:pointer}
.sidebar{width:220px;background:#0f172a;position:fixed;top:0;left:0;height:100vh;padding:20px}
.content{margin-left:250px;padding:20px}
.stat{display:inline-block;background:#2563eb;padding:20px;border-radius:10px;margin:10px;min-width:180px}
table{width:100%;border-collapse:collapse}
th,td{border:1px solid #374151;padding:8px}
.hidden{display:none}
</style>
</head>
<body>

<div id="loginPage" class="login">
<div class="card">
<h1>🏋 Gym Management System</h1>
<p>Admin Login</p>
<input id="user" placeholder="Username" value="admin">
<input id="pass" type="password" placeholder="Password" value="admin123">
<button onclick="login()">Login</button>
<p id="msg"></p>
</div>
</div>

<div id="app" class="hidden">
<div class="sidebar">
<h2>GYM</h2>
<button onclick="show('dashboard')">Dashboard</button><br>
<button onclick="show('members')">Members</button><br>
<button onclick="show('billing')">Billing</button><br>
<button onclick="show('notifications')">Notifications</button><br><br>
<button onclick="logout()">Logout</button>
</div>

<div class="content">
<div id="dashboard">
<h1>Dashboard</h1>
<div class="stat">Members: <span id="memberCount">0</span></div>
<div class="stat">Bills: <span id="billCount">0</span></div>
</div>

<div id="members" class="hidden">
<h2>Members</h2>
<input id="memberName" placeholder="Member Name">
<input id="memberEmail" placeholder="Email">
<button onclick="addMember()">Add Member</button>
<table>
<thead><tr><th>Name</th><th>Email</th><th>Action</th></tr></thead>
<tbody id="memberTable"></tbody>
</table>
</div>

<div id="billing" class="hidden">
<h2>Billing</h2>
<input id="billMember" placeholder="Member Name">
<input id="billAmount" type="number" placeholder="Amount">
<button onclick="addBill()">Create Bill</button>
<table>
<thead><tr><th>Member</th><th>Amount</th></tr></thead>
<tbody id="billTable"></tbody>
</table>
</div>

<div id="notifications" class="hidden">
<h2>Notifications</h2>
<input id="noteText" placeholder="Notification">
<button onclick="addNotification()">Send</button>
<ul id="noteList"></ul>
</div>

</div>
</div>

<script>
let members=JSON.parse(localStorage.getItem('members')||'[]');
let bills=JSON.parse(localStorage.getItem('bills')||'[]');
let notes=JSON.parse(localStorage.getItem('notes')||'[]');

function login(){
 if(user.value==='admin' && pass.value==='admin123'){
  loginPage.classList.add('hidden');
  app.classList.remove('hidden');
  render();
 } else {
  msg.innerText='Invalid Login';
 }
}

function logout(){location.reload();}

function show(id){
 ['dashboard','members','billing','notifications'].forEach(x=>document.getElementById(x).classList.add('hidden'));
 document.getElementById(id).classList.remove('hidden');
}

function addMember(){
 let name=memberName.value.trim();
 let email=memberEmail.value.trim();
 if(!name||!email)return;
 members.push({name,email});
 localStorage.setItem('members',JSON.stringify(members));
 memberName.value='';memberEmail.value='';
 render();
}

function deleteMember(i){
 members.splice(i,1);
 localStorage.setItem('members',JSON.stringify(members));
 render();
}

function addBill(){
 let member=billMember.value;
 let amount=billAmount.value;
 if(!member||!amount)return;
 bills.push({member,amount});
 localStorage.setItem('bills',JSON.stringify(bills));
 render();
}

function addNotification(){
 let t=noteText.value;
 if(!t)return;
 notes.push(t);
 localStorage.setItem('notes',JSON.stringify(notes));
 render();
}

function render(){
 memberCount.innerText=members.length;
 billCount.innerText=bills.length;

 memberTable.innerHTML='';
 members.forEach((m,i)=>{
  memberTable.innerHTML+=`<tr><td>${m.name}</td><td>${m.email}</td><td><button onclick="deleteMember(${i})">Delete</button></td></tr>`;
 });

 billTable.innerHTML='';
 bills.forEach(b=>{
  billTable.innerHTML+=`<tr><td>${b.member}</td><td>₹${b.amount}</td></tr>`;
 });

 noteList.innerHTML='';
 notes.forEach(n=>{
  noteList.innerHTML+=`<li>${n}</li>`;
 });
}
</script>
</body>
</html>
