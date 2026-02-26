# -<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>محطة الفلاح للبترول</title>
<style>
  body { font-family: Arial, sans-serif; margin:0; padding:0; direction: rtl; background:#f0f4f8; }
  header { background: #1976d2; color: white; padding: 1rem; text-align:center; font-size: 1.2rem; font-weight:bold; }
  main { padding: 1rem; max-width: 500px; margin: auto; }
  input, button { width: 100%; padding: 0.6rem; margin: 0.5rem 0; font-size: 1rem; border-radius:5px; border:1px solid #ccc; }
  button { background:#1976d2; color:white; border:none; cursor:pointer; transition: 0.2s; }
  button:hover { background:#1565c0; }
  .hidden { display: none; }
  h2 { color:#1976d2; margin-bottom:0.5rem; }
  #waitingPage { text-align:center; padding:2rem; background:white; border-radius:10px; box-shadow:0 0 10px rgba(0,0,0,0.1); font-size:1.2rem; color:#333; }
  #orderList { list-style: none; padding: 0; }
  #orderList li { background:white; padding:0.5rem; margin-bottom:0.5rem; border-radius:5px; box-shadow:0 1px 3px rgba(0,0,0,0.1); display:flex; justify-content: space-between; align-items:center; }
  #orderList button { width: auto; margin-left:0.5rem; padding:0.3rem 0.5rem; font-size:0.9rem; }
</style>
</head>
<body>

<header>
  مرحبا بك في محطة الفلاح للبترول
</header>

<main>
  <!-- صفحة تسجيل الدخول -->
  <div id="loginPage">
    <h2>تسجيل الدخول</h2>
    <input type="text" id="loginName" placeholder="الاسم">
    <input type="password" id="loginPassword" placeholder="كلمة المرور">
    <button onclick="login()">دخول</button>
    <p>لا تملك حساب؟ <a href="#" onclick="signupPrompt()">إنشاء حساب جديد</a></p>
  </div>

  <!-- صفحة إنشاء الحساب -->
  <div id="signupPage" class="hidden">
    <h2>إنشاء حساب جديد</h2>
    <input type="text" id="signupName" placeholder="الاسم">
    <input type="password" id="signupPassword" placeholder="كلمة المرور">
    <button onclick="signup()">إنشاء الحساب</button>
    <p>لديك حساب بالفعل؟ <a href="#" onclick="showLogin()">تسجيل الدخول</a></p>
  </div>

  <!-- واجهة الزبون -->
  <div id="customerPage" class="hidden">
    <h2>طلب الوقود</h2>
    <select id="fuel">
      <option value="جازولين">جازولين</option>
    </select>
    <input type="number" id="quantity" placeholder="الكمية (لتر)">
    <button onclick="placeOrder()">إرسال الطلب</button>
  </div>

  <!-- صفحة الانتظار للزبون -->
  <div id="waitingPage" class="hidden">
    طلبك قيد الإجراء… يرجى الانتظار
  </div>

  <!-- واجهة السائق -->
  <div id="driverPage" class="hidden">
    <h2>طلبات الوقود</h2>
    <ul id="orderList"></ul>
  </div>
</main>

<script>
// المستخدمون (زبائن فقط) + سائق واحد ثابت
let users = [];
let orders = [];
let currentUser = null;

const driver = { name: "محمد جعفر", password: "0000" };

// صفحات
function signupPrompt() {
  document.getElementById('loginPage').classList.add('hidden');
  document.getElementById('signupPage').classList.remove('hidden');
}
function showLogin() {
  document.getElementById('signupPage').classList.add('hidden');
  document.getElementById('loginPage').classList.remove('hidden');
}

// إنشاء حساب زبون
function signup() {
  const name = document.getElementById('signupName').value.trim();
  const password = document.getElementById('signupPassword').value.trim();
  if(!name || !password){ alert("الرجاء إدخال الاسم وكلمة المرور"); return; }
  users.push({ name, password });
  alert("تم إنشاء الحساب بنجاح!");
  showLogin();
}

// تسجيل الدخول (زبون أو السائق)
function login() {
  const name = document.getElementById('loginName').value.trim();
  const password = document.getElementById('loginPassword').value.trim();

  if(name === driver.name && password === driver.password){
    currentUser = driver;
    document.getElementById('loginPage').classList.add('hidden');
    document.getElementById('driverPage').classList.remove('hidden');
    updateDriverOrders();
    return;
  }

  const user = users.find(u => u.name === name && u.password === password);
  if(!user){ alert("الاسم أو كلمة المرور غير صحيحة"); return; }
  currentUser = user;
  document.getElementById('loginPage').classList.add('hidden');
  document.getElementById('customerPage').classList.remove('hidden');
}

// الزبون يرسل الطلب وتظهر صفحة الانتظار
function placeOrder() {
  const fuel = document.getElementById('fuel').value;
  const quantity = document.getElementById('quantity').value;
  if(!quantity || quantity<=0){ alert("الرجاء إدخال كمية صحيحة"); return; }

  const order = {
    customer: currentUser.name,
    fuel,
    quantity,
    status: "pending"
  };
  orders.push(order);

  // إخفاء صفحة الطلب وإظهار صفحة الانتظار
  document.getElementById('customerPage').classList.add('hidden');
  document.getElementById('waitingPage').classList.remove('hidden');

  updateDriverOrders();
}

// تحديث الطلبات عند السائق
function updateDriverOrders() {
  const list = document.getElementById('orderList');
  list.innerHTML = "";
  orders.forEach((order,index) => {
    if(order.status === "pending") {
      const li = document.createElement('li');
      li.textContent = `${order.customer} طلب ${order.quantity} لتر من ${order.fuel}`;
      const confirmBtn = document.createElement('button');
      confirmBtn.textContent = "تأكيد الطلب";
      confirmBtn.onclick = () => {
        order.status = "confirmed";
        alert(`الزبون ${order.customer}: الطلب في الطريق إليك`);
        updateDriverOrders();
      };
      const stopBtn = document.createElement('button');
      stopBtn.textContent = "المحطة متوقفة";
      stopBtn.onclick = () => {
        order.status = "cancelled";
        alert(`الزبون ${order.customer}: عذرًا، يرجى الطلب لاحقًا`);
        updateDriverOrders();
      };
      li.appendChild(confirmBtn);
      li.appendChild(stopBtn);
      list.appendChild(li);
    }
  });
}
</script>

</body>
</html>
