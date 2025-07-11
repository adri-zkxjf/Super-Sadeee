<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>IperShop</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body {
      font-family: 'Poppins', sans-serif;
      background: #f0f4f8;
      margin: 0;
      padding: 0;
      color: #333;
    }

    .container, .login-container, .admin-panel {
      max-width: 400px;
      margin: 50px auto;
      padding: 20px;
      background: #fff;
      border-radius: 12px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.1);
    }

    input[type="email"], input[type="password"], input[type="text"] {
      width: 100%;
      padding: 10px;
      margin: 8px 0;
      border-radius: 8px;
      border: 1px solid #ccc;
      box-sizing: border-box;
    }

    button {
      background: #1e90ff;
      color: white;
      border: none;
      padding: 10px 15px;
      border-radius: 8px;
      cursor: pointer;
      margin-top: 10px;
      width: 100%;
    }

    button:hover {
      background: #0077cc;
    }

    .popens-box {
      background: #ffcc00;
      padding: 10px;
      border-radius: 8px;
      text-align: center;
      margin-bottom: 15px;
      font-weight: bold;
    }

    ul {
      list-style: none;
      padding: 0;
      max-height: 150px;
      overflow-y: auto;
    }

    li {
      background: #f9f9f9;
      margin: 5px 0;
      padding: 8px;
      border-radius: 6px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .recommendation-box, .pdf-link-box {
      margin-top: 15px;
      padding: 10px;
      border-radius: 8px;
      background: #e0f7fa;
      color: #00796b;
    }

    .timer-box {
      background: #ffe0e0;
      padding: 10px;
      border-radius: 8px;
      text-align: center;
      margin-top: 10px;
      color: #c62828;
    }
  </style>
</head>
<body>

<div class="login-container" id="loginScreen">
  <h2>🔐 Inicia Sesión</h2>
  <input type="email" id="loginEmail" placeholder="Gmail">
  <input type="password" id="loginPassword" placeholder="Contraseña">
  <button onclick="login()">Iniciar Sesión</button>
  <p>¿No tienes cuenta? <a href="#" onclick="showRegister()">Regístrate</a></p>
</div>

<div class="login-container" id="registerScreen" style="display:none;">
  <h2>📝 Registro</h2>
  <input type="email" id="registerEmail" placeholder="Gmail">
  <input type="password" id="registerPass" placeholder="Contraseña">
  <input type="password" id="registerPassConfirm" placeholder="Confirmar Contraseña">
  <button onclick="register()">Registrarse</button>
  <p>¿Ya tienes cuenta? <a href="#" onclick="showLogin()">Inicia sesión</a></p>
</div>

<div class="admin-panel" id="adminPanel" style="display:none;">
  <h2>🔧 Panel de Control (Admin)</h2>
  <div id="adminUsers"></div>
  <button onclick="showApp()">Ir a la App 🛒</button>
  <button onclick="logout()">Cerrar Sesión</button>
</div>

<div class="container" id="mainApp" style="display:none;">
  <h1>🛒 Lista de la Compra Inteligente</h1>

  <div class="popens-box">
    💰 Popens: <span id="popensCount">100000</span>
  </div>

  <div class="input-group">
    <input id="productInput" type="text" placeholder="Producto (ej: Yogur)">
    <input id="brandInput" type="text" placeholder="Marca (ej: Larsa)">
    <input id="locationInput" type="text" placeholder="Ubicación">
    <button onclick="addProduct()">➕ Añadir</button>
  </div>

  <ul id="productList"></ul>

  <button onclick="showRecommendations()">🔍 Ver Recomendación</button>
  <div id="recommendation" class="recommendation-box"></div>

  <div id="pdfLinkBox" class="pdf-link-box" style="display:none;">
    📄 <a id="downloadPdfLink" href="#" download="compra.pdf">Descargar PDF de la compra</a>
  </div>

  <div id="timerBox" class="timer-box" style="display:none;">
    ⏳ Espera <span id="timerCountdown">24:00:00</span>
  </div>

  <div id="adminReturn" style="display:none;">
    <button onclick="promptAdminReturn()">🔑 Volver a Panel Admin</button>
  </div>

  <button onclick="logout()">Cerrar Sesión</button>
</div>

<script>
  const productDatabase = {
    'agua mineral': { marca: 'Fontecabras', supermercado: 'Mercadona', precio: 0.66 },
    'pan de barra': { marca: 'Hacendado', supermercado: 'Mercadona', precio: 0.85 },
    'pan de molde': { marca: 'Bimbo', supermercado: 'Lidl', precio: 1.50 },
    'leche entera': { marca: 'Pascual', supermercado: 'Carrefour', precio: 1.20 },
    'huevos': { marca: 'Campesino', supermercado: 'Mercadona', precio: 2.80 },
    'aceite de oliva': { marca: 'Carbonell', supermercado: 'Carrefour', precio: 8.99 },
    'arroz': { marca: 'Brillante', supermercado: 'Mercadona', precio: 2.60 },
    'pasta': { marca: 'Gallo', supermercado: 'Mercadona', precio: 1.20 },
    'azucar': { marca: 'Hacendado', supermercado: 'Mercadona', precio: 1.00 },
    'manzanas': { marca: 'Royal Gala', supermercado: 'Lidl', precio: 2.20 },
    'platano': { marca: 'Marca Blanca', supermercado: 'Lidl', precio: 1.75 },
    'ternera': { marca: 'Fresca', supermercado: 'Carrefour', precio: 14.00 }
  };

  let users = JSON.parse(localStorage.getItem('users') || '{}');
  let currentUser = null;
  let isAdmin = false;
  let popens = 100000;
  let products = [];
  let timerInterval;

  function showLogin() {
    hideAll();
    document.getElementById('loginScreen').style.display = 'block';
  }

  function showRegister() {
    hideAll();
    document.getElementById('registerScreen').style.display = 'block';
  }

  function showAdmin() {
    hideAll();
    document.getElementById('adminPanel').style.display = 'block';
    renderAdmin();
  }

  function showApp() {
    hideAll();
    document.getElementById('mainApp').style.display = 'block';
    document.getElementById('adminReturn').style.display = isAdmin ? 'block' : 'none';
    updatePopens();
    renderProducts();
    checkTimerOnLoad();
  }

  function hideAll() {
    document.querySelectorAll('body > div').forEach(div => div.style.display = 'none');
  }

  function login() {
    const email = document.getElementById('loginEmail').value.trim().toLowerCase();
    const pass = document.getElementById('loginPassword').value;

    if (email === 'adriangonzalezmiguens@gmail.com' && pass === 'adrigm2011') {
      currentUser = email;
      isAdmin = true;
      showAdmin();
      return;
    }

    const user = users[email];
    if (user && user.pass === pass) {
      if (user.blocked) return alert('Acceso bloqueado.');
      currentUser = email;
      isAdmin = false;
      popens = user.popens ?? 100000;
      products = [];
      showApp();
    } else {
      alert('Credenciales incorrectas.');
    }
  }

  function register() {
    const email = document.getElementById('registerEmail').value.trim().toLowerCase();
    const pass = document.getElementById('registerPass').value;
    const confirm = document.getElementById('registerPassConfirm').value;

    if (!email || !pass || pass !== confirm) {
      alert('Verifica los campos.');
      return;
    }

    if (users[email]) {
      alert('Usuario ya existe.');
      return;
    }

    users[email] = { pass, popens: 100000, blocked: false, searches: [] };
    localStorage.setItem('users', JSON.stringify(users));
    alert('Registro exitoso.');
    showLogin();
  }

  function logout() {
    saveUserData();
    currentUser = null;
    isAdmin = false;
    products = [];
    popens = 100000;
    clearInterval(timerInterval);
    document.getElementById('timerBox').style.display = 'none';
    showLogin();
  }

  function renderAdmin() {
    const adminUsers = document.getElementById('adminUsers');
    adminUsers.innerHTML = '<h3>Usuarios Registrados:</h3>';
    Object.keys(users).forEach(email => {
      const user = users[email];
      adminUsers.innerHTML += `
        <div>
          📧 ${email} | 🔑 ${user.pass} | 💰 ${user.popens ?? 0} | 
          ${user.blocked ? '🚫 Bloqueado' : '✅ Activo'}<br>
          <button onclick="toggleBlock('${email}')">${user.blocked ? 'Desbloquear' : 'Bloquear'}</button>
          <button onclick="promptGiftPopens('${email}')">Regalar Popens</button>
        </div>
      `;
    });
  }

  function toggleBlock(email) {
    users[email].blocked = !users[email].blocked;
    localStorage.setItem('users', JSON.stringify(users));
    renderAdmin();
  }

  function promptGiftPopens(email) {
    const amount = parseInt(prompt('¿Cuántos Popens quieres regalar?'), 10);
    if (!isNaN(amount) && amount > 0) {
      users[email].popens = (users[email].popens || 0) + amount;
      localStorage.setItem('users', JSON.stringify(users));
      renderAdmin();
    }
  }

  function addProduct() {
    if (popens < 20000) {
      alert('Sin Popens.');
      return;
    }

    const p = document.getElementById('productInput').value.trim().toLowerCase();
    const b = document.getElementById('brandInput').value.trim();
    const l = document.getElementById('locationInput').value.trim();

    if (!p || !b || !l) {
      alert('Completa todos los campos.');
      return;
    }

    products.push({ producto: p, marca: b, lugar: l });
    popens -= 20000;

    saveUserData();
    updatePopens();
    renderProducts();
  }

  function renderProducts() {
    const list = document.getElementById('productList');
    list.innerHTML = '';

    products.forEach((item, index) => {
      list.innerHTML += `<li>${item.producto} - ${item.marca} - ${item.lugar} 
        <button onclick="removeProduct(${index})">❌</button></li>`;
    });

    if (products.length >= 2) {
      generatePDFLink();
    } else {
      document.getElementById('pdfLinkBox').style.display = 'none';
    }
  }

  function removeProduct(index) {
    products.splice(index, 1);
    renderProducts();
  }

  function updatePopens() {
    document.getElementById('popensCount').innerText = popens;
    if (popens <= 0) startTimer();
  }

  function saveUserData() {
    if (currentUser && users[currentUser]) {
      users[currentUser].popens = popens;
      localStorage.setItem('users', JSON.stringify(users));
    }
  }

  function showRecommendations() {
    if (products.length === 0) {
      alert('Añade productos.');
      return;
    }

    const last = products[products.length - 1].producto;
    const data = productDatabase[last] || { supermercado: 'Carrefour', marca: 'Genérica', precio: 1.00 };
    const link = `https://www.google.com/maps/search/${encodeURIComponent(data.supermercado)}`;

    document.getElementById('recommendation').innerHTML = `
      Mejor: <strong>${data.supermercado}</strong> para <strong>${last}</strong><br>
      <a href="${link}" target="_blank">📍 Cómo llegar</a>
    `;
  }

  function startTimer() {
    document.getElementById('timerBox').style.display = 'block';
    let timeLeft = 24 * 60 * 60;

    clearInterval(timerInterval);
    timerInterval = setInterval(() => {
      timeLeft--;
      const h = String(Math.floor(timeLeft / 3600)).padStart(2, '0');
      const m = String(Math.floor((timeLeft % 3600) / 60)).padStart(2, '0');
      const s = String(timeLeft % 60).padStart(2, '0');
      document.getElementById('timerCountdown').innerText = `${h}:${m}:${s}`;

      if (timeLeft <= 0) {
        clearInterval(timerInterval);
        popens = 100000;
        updatePopens();
        document.getElementById('timerBox').style.display = 'none';
      }
    }, 1000);
  }

  function checkTimerOnLoad() {
    if (popens <= 0) startTimer();
  }

  function promptAdminReturn() {
    const pass = prompt('Contraseña admin:');
    if (pass === 'adrigm2011') showAdmin();
  }

  function generatePDFLink() {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    let y = 20;

    // Marca de agua "IperShop"
    doc.setTextColor(230);
    doc.setFontSize(60);
    doc.text('IperShop', 35, 150, { angle: 45 });

    doc.setTextColor(0);
    doc.setFontSize(16);
    doc.text('🛒 Lista de la Compra', 10, y);
    y += 10;

    doc.setFontSize(12);
    doc.text('Producto | Marca | Supermercado | Precio (€)', 10, y);
    y += 10;

    const totals = {};

    products.forEach(item => {
      const db = productDatabase[item.producto] || { marca: item.marca, supermercado: 'Carrefour', precio: 1.00 };
      const linea = `${capitalize(item.producto)} | ${db.marca} | ${db.supermercado} | ${db.precio.toFixed(2)}`;
      doc.text(linea, 10, y);
      y += 8;

      totals[db.supermercado] = (totals[db.supermercado] || 0) + db.precio;
    });

    const mejorSuper = Object.keys(totals).reduce((a, b) => totals[a] < totals[b] ? a : b);

    y += 10;
    doc.setFontSize(14);
    doc.text(`✅ Mejor opción: ${mejorSuper} con un total de €${totals[mejorSuper].toFixed(2)}`, 10, y);

    const pdfBlob = doc.output('blob');
    const pdfUrl = URL.createObjectURL(pdfBlob);
    const link = document.getElementById('downloadPdfLink');
    link.href = pdfUrl;

    document.getElementById('pdfLinkBox').style.display = 'block';
  }

  function capitalize(str) {
    return str.charAt(0).toUpperCase() + str.slice(1);
  }

  showLogin();
</script>

</body>
</html>
