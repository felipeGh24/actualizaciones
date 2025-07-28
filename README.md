# actualizaciones
a actualización de la página web de una tienda de comercio electrónico con la finalidad de permitir a los usuarios buscar y filtrar productos, agregarlos a su carrito de compras, realizar pagos en línea de manera segura y recibir notificaciones sobre promociones y descuentos.
<!-- index.html -->
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Tienda Online</title>
  <link rel="stylesheet" href="styles.css" />
</head>
<body>
  <h1>Tienda Electrónica</h1>

  <input type="text" id="busqueda" placeholder="Buscar productos..." />
  <select id="categoria">
    <option value="">Todas</option>
    <option value="laptop">Laptops</option>
    <option value="telefono">Teléfonos</option>
  </select>

  <div id="productos"></div>

  <h2>Carrito de Compras</h2>
  <ul id="carrito"></ul>
  <button onclick="pagar()">Realizar Pago</button>

  <div id="notificacion"></div>

  <script src="app.js"></script>
</body>
</html>

/* styles.css */
body {
  font-family: Arial, sans-serif;
  padding: 20px;
  background: #f0f0f0;
}

#productos {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
}

.producto {
  background: white;
  padding: 10px;
  border-radius: 8px;
  box-shadow: 0 2px 5px #ccc;
}

#notificacion {
  margin-top: 20px;
  padding: 10px;
  background: #d4edda;
  color: #155724;
  border-radius: 4px;
  display: none;
}

// app.js
document.addEventListener("DOMContentLoaded", () => {
  cargarProductos();

  document.getElementById("busqueda").addEventListener("input", cargarProductos);
  document.getElementById("categoria").addEventListener("change", cargarProductos);
});

let carrito = [];

function cargarProductos() {
  const busqueda = document.getElementById("busqueda").value.toLowerCase();
  const categoria = document.getElementById("categoria").value;

  fetch("/api/productos")
    .then(res => res.json())
    .then(data => {
      let filtrados = data.filter(p => 
        p.nombre.toLowerCase().includes(busqueda) &&
        (categoria === "" || p.categoria === categoria)
      );

      mostrarProductos(filtrados);
    });
}

function mostrarProductos(productos) {
  const contenedor = document.getElementById("productos");
  contenedor.innerHTML = "";

  productos.forEach(p => {
    const div = document.createElement("div");
    div.className = "producto";
    div.innerHTML = `
      <h3>${p.nombre}</h3>
      <p>${p.descripcion}</p>
      <p><strong>$${p.precio}</strong></p>
      <button onclick='agregarAlCarrito(${JSON.stringify(p)})'>Agregar</button>
    `;
    contenedor.appendChild(div);
  });
}

function agregarAlCarrito(producto) {
  carrito.push(producto);
  actualizarCarrito();
}

function actualizarCarrito() {
  const ul = document.getElementById("carrito");
  ul.innerHTML = "";
  carrito.forEach((p, i) => {
    const li = document.createElement("li");
    li.textContent = `${p.nombre} - $${p.precio}`;
    ul.appendChild(li);
  });
}

function pagar() {
  const total = carrito.reduce((sum, p) => sum + p.precio, 0);
  alert(`Pago procesado por $${total}. ¡Gracias por su compra!`);
  carrito = [];
  actualizarCarrito();
  mostrarNotificacion("Compra exitosa. ¡No olvides revisar nuestras promociones!");
}

function mostrarNotificacion(msg) {
  const n = document.getElementById("notificacion");
  n.textContent = msg;
  n.style.display = "block";
  setTimeout(() => n.style.display = "none", 4000);
}

// server.js
const express = require('express');
const cors = require('cors');
const app = express();
app.use(cors());
app.use(express.static('public'));

const productos = [
  { id: 1, nombre: 'Laptop HP', descripcion: 'Intel i5 8GB RAM', precio: 450000, categoria: 'laptop' },
  { id: 2, nombre: 'iPhone 12', descripcion: '128GB - Negro', precio: 699000, categoria: 'telefono' },
  { id: 3, nombre: 'Samsung Galaxy A54', descripcion: 'Android 128GB', precio: 300000, categoria: 'telefono' },
  { id: 4, nombre: 'Lenovo IdeaPad', descripcion: 'Ryzen 5, SSD 512GB', precio: 520000, categoria: 'laptop' },
];

app.get('/api/productos', (req, res) => {
  res.json(productos);
});

const PORT = 3000;
app.listen(PORT, () => console.log(`Servidor en http://localhost:${PORT}`));
