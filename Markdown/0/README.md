
### LEER


### CONVERSACIONES CHAPY


### COPY PAST WHATEVER

## IMAGES
![]([Markdown/0/IMG_0505.jpeg](https://github.com/Jordanbuco/Drive/blob/b451b58ce2cba711b0969118337921d2f5258784/Markdown/0/IMG_0505.jpeg)){width='100px'}

### index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mi Aplicación</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div id="app">
        <!-- Navigation Links -->
        <nav>
            <button onclick="showView('home')">INICIO</button>
            <button onclick="showView('pablo')">PABLO</button>
            <button onclick="showView('manolo')">MANOLO</button>
        </nav>
        
        <!-- Home View -->
        <div id="homeView" class="view">
            <h1>INICIO</h1>
        </div>

        <!-- Pablo View -->
        <div id="pabloView" class="view">
            <h1>Dashboard de Pablo</h1>
            <button onclick="loadRooms()">Cargar habitaciones</button>
            <div id="roomsPablo"></div>
        </div>

        <!-- Manolo View -->
        <div id="manoloView" class="view">
            <h1>Dashboard de Manolo</h1>
            <h2>Habitaciones Disponibles:</h2>
            <div id="roomsManolo"></div>
        </div>
    </div>

    <script src="script.js"></script>
</body>
</html>
```
styles.css
```
body {
    font-family: Arial, sans-serif;
    background-color: #06214D;
    color: white;
    margin: 0;
    padding: 0;
}

nav {
    text-align: center;
    margin: 20px;
}

button {
    background-color: #0056b3;
    color: white;
    padding: 10px 20px;
    margin: 10px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
}

button:hover {
    background-color: #004094;
}

.view {
    display: none;
    padding: 20px;
}

.view.active {
    display: block;
}

h1 {
    color: #00BFFF;
}

h2 {
    color: #FFD700;
}

.room {
    border: 1px solid #00BFFF;
    padding: 10px;
    margin: 10px 0;
    border-radius: 5px;
}
```
script.js
```
document.addEventListener("DOMContentLoaded", function() {
    showView('home');
});

function showView(viewId) {
    // Ocultar todas las vistas
    document.querySelectorAll('.view').forEach(view => view.classList.remove('active'));
    // Mostrar la vista seleccionada
    document.getElementById(viewId + 'View').classList.add('active');
}

function loadRooms() {
    const url = 'https://raw.githubusercontent.com/Jordanbuco/Data/main/hostman/db.json';
    
    fetch(url)
        .then(response => response.json())
        .then(data => {
            const availableRooms = data.filter(room => room.dispo);
            displayRooms(availableRooms, 'roomsPablo');
            displayRooms(availableRooms, 'roomsManolo');
        })
        .catch(error => console.error('Error al cargar las habitaciones:', error));
}

function displayRooms(rooms, containerId) {
    const container = document.getElementById(containerId);
    container.innerHTML = ''; // Limpiar contenido previo

    rooms.forEach(room => {
        const roomDiv = document.createElement('div');
        roomDiv.classList.add('room');
        roomDiv.innerHTML = `
            <h3>${room.title}</h3>
            <p>${room.body}</p>
            <p>Precio: $${room.price.toFixed(2)}</p>
        `;
        container.appendChild(roomDiv);
    });
}
```
