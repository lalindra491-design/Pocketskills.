# Pocketskills.
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pocket Skills - Pro SaaS</title>

<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;600&display=swap" rel="stylesheet">

<style>
body{margin:0;font-family:Poppins;background:#020c1b;color:#fff;}
canvas{position:fixed;top:0;left:0;z-index:0;}
nav{position:fixed;width:100%;display:flex;justify-content:space-between;padding:15px;background:rgba(0,0,0,0.6);z-index:10;}
.container{position:relative;z-index:1;padding:100px 20px;max-width:1000px;margin:auto;}
.card{background:rgba(255,255,255,0.05);padding:20px;margin:15px 0;border-radius:12px;}
button{background:#00e0ff;border:none;padding:10px;margin:5px;cursor:pointer;}
input{width:100%;padding:10px;margin:10px 0;border:none;border-radius:5px;}
.hidden{display:none;}
video{width:100%;border-radius:10px;}
</style>
</head>

<body>

<canvas id="bg"></canvas>

<nav>
<h2>Pocket Skills</h2>
<button onclick="logout()">Logout</button>
</nav>

<div class="container">

<!-- AUTH -->
<div id="auth" class="card">
<h2>Login / Register</h2>
<input type="email" id="email" placeholder="Email">
<input type="password" id="password" placeholder="Password">
<button onclick="register()">Register</button>
<button onclick="login()">Login</button>
</div>

<!-- USER DASHBOARD -->
<div id="dashboard" class="hidden">

<div class="card">
<h3>Referral Link</h3>
<p id="refLink"></p>
<button onclick="copyLink()">Copy</button>
</div>

<div class="card">
<h3>Earnings</h3>
<p id="earnings">₹0</p>
</div>

<div class="card">
<h3>Payment ₹499</h3>
<img src="qr.png" width="200">
<p>Scan → Pay → Wait for approval</p>
</div>

<div class="card">
<h3>Course</h3>
<div id="courseArea">Locked</div>
</div>

</div>

<!-- ADMIN PANEL -->
<div id="admin" class="hidden">
<div class="card">
<h2>Admin Panel</h2>
<div id="users"></div>
</div>
</div>

</div>

<!-- THREE JS -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

<script>
// 3D BACKGROUND
const scene=new THREE.Scene();
const camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,0.1,1000);
const renderer=new THREE.WebGLRenderer({canvas:bg});
renderer.setSize(innerWidth,innerHeight);

const geo=new THREE.TorusKnotGeometry(10,3,100,16);
const mat=new THREE.MeshBasicMaterial({color:0x00e0ff,wireframe:true});
const mesh=new THREE.Mesh(geo,mat);
scene.add(mesh);

camera.position.z=30;

function animate(){
requestAnimationFrame(animate);
mesh.rotation.x+=0.01;
mesh.rotation.y+=0.01;
renderer.render(scene,camera);
}
animate();

// FIREBASE CONFIG
const firebaseConfig={
apiKey:"YOUR_API_KEY",
authDomain:"YOUR_DOMAIN",
projectId:"YOUR_PROJECT_ID"
};

firebase.initializeApp(firebaseConfig);
const auth=firebase.auth();
const db=firebase.firestore();

// REGISTER
function register(){
let email=document.getElementById("email").value;
let password=document.getElementById("password").value;

auth.createUserWithEmailAndPassword(email,password)
.then(user=>{
db.collection("users").doc(user.user.uid).set({
email:email,
earnings:0,
course:false
});
alert("Registered");
});
}

// LOGIN
function login(){
auth.signInWithEmailAndPassword(
document.getElementById("email").value,
document.getElementById("password").value
);
}

// AUTH STATE
auth.onAuthStateChanged(user=>{
if(user){

// ADMIN
if(user.email==="admin@pocketskills.com"){
showAdmin();
loadUsers();
return;
}

// USER
showDashboard();
let uid=user.uid;

refLink.innerText=location.href+"?ref="+uid;

db.collection("users").doc(uid).get().then(doc=>{
let d=doc.data();
earnings.innerText="₹"+d.earnings;

if(d.course){
courseArea.innerHTML=`<video controls src="https://www.w3schools.com/html/mov_bbb.mp4"></video>`;
}
});

}else{
showAuth();
}
});

// UI CONTROL
function showAuth(){
authDiv.classList.remove("hidden");
dashboard.classList.add("hidden");
admin.classList.add("hidden");
}
function showDashboard(){
authDiv.classList.add("hidden");
dashboard.classList.remove("hidden");
admin.classList.add("hidden");
}
function showAdmin(){
authDiv.classList.add("hidden");
dashboard.classList.add("hidden");
admin.classList.remove("hidden");
}

// REF SYSTEM
const params=new URLSearchParams(location.search);
if(params.get("ref")){
localStorage.setItem("referrer",params.get("ref"));
}

// COPY
function copyLink(){
navigator.clipboard.writeText(refLink.innerText);
alert("Copied");
}

// ADMIN LOAD USERS
function loadUsers(){
db.collection("users").get().then(snapshot=>{
let html="";
snapshot.forEach(doc=>{
let d=doc.data();
html+=`
<div class="card">
${d.email}<br>
Earnings: ₹${d.earnings}<br>
<button onclick="approve('${doc.id}')">Approve Course</button>
</div>`;
});
users.innerHTML=html;
});
}

// APPROVE
function approve(uid){
db.collection("users").doc(uid).update({course:true});
alert("Approved");
}

// LOGOUT
function logout(){
auth.signOut();
}
</script>

<!-- FIREBASE -->
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-auth.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js"></script>

</body>
</html>
