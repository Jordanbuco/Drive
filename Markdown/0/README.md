
# LEER



# COPY PAST WHATEVER



# IMAGES
## Galería de Imágenes
### Imagen Principal

<div style="display: flex; gap: 10px;">
  <img src="https://github.com/Jordanbuco/Drive/blob/main/Markdown/0/imgs/IMG_0506.jpeg" alt="Imagen 1 - Miniatura" width="100"/>
  <img src="https://github.com/Jordanbuco/Drive/blob/main/Markdown/0/imgs/IMG_0507.jpeg" alt="Imagen 2 - Miniatura" width="100"/>
  
</div>

# CONVERSACIONES CHAPY
## CODIGOS **PASTE** DE CONVERSACION CON **CHAPY**
### **PREGUNTA**
### *PASAME ESTE CODIGO DE SWIFT PLAYGROUND DE IPAD A HTML CSS Y JS*
## <CODE>
```
import SwiftUI
import WebKit
import Foundation

// MARK: - Modelos

struct Room: Identifiable, Codable {
    let userId: Int
    let id: Int
    let title: String
    let body: String
    let price: Double
    let dispo: Bool
    let images: String
}

struct Hostal {
    let nombre: String
    let habitaciones: [Room]
}

// MARK: - Gestor de Habitaciones

class RoomManager: ObservableObject {
    @Published var habitaciones: [Room] = []
    private let url = "https://raw.githubusercontent.com/Jordanbuco/Data/main/hostman/db.json"
    private let token = "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"
    
    static let shared = RoomManager()
    
    init() {
        loadRooms()
    }
    
    func loadRooms() {
        NetworkManager.shared.fetchRooms(from: url) { rooms in
            DispatchQueue.main.async {
                self.habitaciones = rooms ?? []
            }
        }
    }
    
    func saveRooms() {
        let message = "Actualización del archivo JSON con nuevas habitaciones"
        GitHubAPIManager().updateRooms(habitaciones, message: message) { result in
            switch result {
            case .success:
                print("Habitaciones guardadas exitosamente en GitHub")
            case .failure(let error):
                print("Error al guardar habitaciones en GitHub: \(error.localizedDescription)")
            }
        }
    }
}

// MARK: - Operaciones de Habitaciones

class RoomOperations {
    static let shared = RoomOperations()
    
    func addRoom(_ room: Room, to rooms: inout [Room]) {
        rooms.append(room)
        RoomManager.shared.saveRooms()
    }
    
    func deleteRoom(byId id: Int, from rooms: inout [Room]) {
        rooms.removeAll { $0.id == id }
        RoomManager.shared.saveRooms()
    }
    
    func editRoom(_ updatedRoom: Room, in rooms: inout [Room]) {
        if let index = rooms.firstIndex(where: { $0.id == updatedRoom.id }) {
            rooms[index] = updatedRoom
            RoomManager.shared.saveRooms()
        }
    }
}

// MARK: - Gestor de la API de GitHub

class GitHubAPIManager {
    private let token = "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"
    private let repo = "Data"
    private let owner = "Jordanbuco"
    private let jsonFilePath = "path/to/your/rooms.json"
    
    func fetchRooms(completion: @escaping (Result<[Room], Error>) -> Void) {
        let url = URL(string: "https://raw.githubusercontent.com/\(owner)/\(repo)/main/\(jsonFilePath)")!
        
        let task = URLSession.shared.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(NSError(domain: "", code: -1, userInfo: [NSLocalizedDescriptionKey: "No data"])))
                return
            }
            
            do {
                let rooms = try JSONDecoder().decode([Room].self, from: data)
                completion(.success(rooms))
            } catch {
                completion(.failure(error))
            }
        }
        task.resume()
    }
    
    func updateRooms(_ rooms: [Room], message: String, completion: @escaping (Result<Void, Error>) -> Void) {
        let url = URL(string: "https://api.github.com/repos/\(owner)/\(repo)/contents/\(jsonFilePath)")!
        
        do {
            let jsonData = try JSONEncoder().encode(rooms)
            let base64Content = jsonData.base64EncodedString()
            
            var request = URLRequest(url: url)
            request.httpMethod = "GET"
            request.setValue("token \(token)", forHTTPHeaderField: "Authorization")
            
            let task = URLSession.shared.dataTask(with: request) { data, response, error in
                guard let data = data, error == nil else {
                    completion(.failure(error ?? NSError(domain: "", code: -1, userInfo: [NSLocalizedDescriptionKey: "Network error"])))
                    return
                }
                
                do {
                    if let jsonResponse = try JSONSerialization.jsonObject(with: data, options: []) as? [String: Any],
                       let sha = jsonResponse["sha"] as? String {
                        
                        let updatePayload: [String: Any] = [
                            "message": message,
                            "content": base64Content,
                            "sha": sha
                        ]
                        let payloadData = try JSONSerialization.data(withJSONObject: updatePayload, options: [])
                        
                        var putRequest = URLRequest(url: url)
                        putRequest.httpMethod = "PUT"
                        putRequest.setValue("token \(self.token)", forHTTPHeaderField: "Authorization")
                        putRequest.setValue("application/json", forHTTPHeaderField: "Content-Type")
                        putRequest.httpBody = payloadData
                        
                        let putTask = URLSession.shared.dataTask(with: putRequest) { data, response, error in
                            if let error = error {
                                completion(.failure(error))
                            } else {
                                completion(.success(()))
                            }
                        }
                        putTask.resume()
                        
                    } else {
                        completion(.failure(NSError(domain: "", code: -1, userInfo: [NSLocalizedDescriptionKey: "Failed to parse response"])))
                    }
                } catch {
                    completion(.failure(error))
                }
            }
            task.resume()
            
        } catch {
            completion(.failure(error))
        }
    }
}

// MARK: - Network Manager

class NetworkManager {
    static let shared = NetworkManager()
    
    func fetchRooms(from urlString: String, completion: @escaping ([Room]?) -> Void) {
        guard let url = URL(string: urlString) else {
            completion(nil)
            return
        }
        
        let task = URLSession.shared.dataTask(with: url) { data, response, error in
            if let data = data {
                let rooms = try? JSONDecoder().decode([Room].self, from: data)
                completion(rooms)
            } else {
                completion(nil)
            }
        }
        task.resume()
    }
}

// MARK: - Vistas

struct ContentView: View {
    @ObservedObject var roomManager: RoomManager
    
    var body: some View {
        NavigationView {
            GeometryReader { geometry in
                ZStack {
                    Color(.sRGB, red: 6/255, green: 33/255, blue: 77/255, opacity: 1)
                        .ignoresSafeArea()
                    
                    VStack(spacing: 20) {
                        Text("INICIO")
                            .font(.system(size: geometry.size.width * 0.1))
                            .fontWeight(.bold)
                            .foregroundColor(.blue)
                            .padding()
                            .frame(width: 200)
                        
                        NavigationLink(destination: PabloView(roomManager: roomManager)) {
                            Text("PABLO")
                                .font(.system(size: geometry.size.width * 0.05))
                                .fontWeight(.light)
                                .padding()
                                .frame(width: 200)
                                .background(Color.blue)
                                .foregroundColor(.white)
                                .cornerRadius(10)
                                .overlay(
                                    RoundedRectangle(cornerRadius: 10)
                                        .stroke(Color.yellow, lineWidth: 2)
                                )
                        }
                        
                        NavigationLink(destination: ManoloView(roomManager: roomManager)) {
                            Text("MANOLO")
                                .font(.system(size: geometry.size.width * 0.05))
                                .fontWeight(.light)
                                .padding()
                                .frame(width: 200)
                                .background(Color.blue)
                                .foregroundColor(.white)
                                .cornerRadius(10)
                                .overlay(
                                    RoundedRectangle(cornerRadius: 10)
                                        .stroke(Color.yellow, lineWidth: 2)
                                )
                        }
                    }
                }
            }
        }
    }
}

struct PabloView: View {
    @ObservedObject var roomManager: RoomManager
    
    var body: some View {
        GeometryReader { geometry in
            ZStack {
                Color(.sRGB, red: 6/255, green: 33/255, blue: 77/255, opacity: 1)
                    .ignoresSafeArea()
                
                VStack {
                    Text("Dashboard de:")
                        .font(.system(size: geometry.size.width * 0.05))
                        .padding()
                        .foregroundColor(.cyan)
                    
                    NavigationLink(destination: BaseView()) {
                        Text("PABLO")
                            .font(.system(size: geometry.size.width * 0.06))
                            .fontWeight(.bold)
                            .cornerRadius(10)
                            .padding(15)
                            .foregroundColor(.blue)
                            .overlay(
                                RoundedRectangle(cornerRadius: 10)
                                    .stroke(Color.yellow, lineWidth: 1)
                            )
                    }
                    
                    Text("Información del Dashboard")
                        .font(.system(size: geometry.size.width * 0.05))
                        .padding(25)
                        .foregroundColor(.cyan)
                    
                    Button(action: {
                        roomManager.loadRooms()
                    }) {
                        Text("Cargar habitaciones")
                            .font(.system(size: geometry.size.width * 0.05))
                            .foregroundColor(.white)
                            .padding()
                            .background(Color.blue)
                            .cornerRadius(10)
                    }
                }
            }
        }
    }
}

struct ManoloView: View {
    @ObservedObject var roomManager: RoomManager
    
    var body: some View {
        ZStack {
            Color(.sRGB, red: 6/255, green: 33/255, blue: 77/255, opacity: 1)
                .ignoresSafeArea()
            
            VStack {
                Text("Dashboard de Manolo")
                    .font(.largeTitle)
                    .padding()
                    .foregroundColor(.cyan)
                
                Text("Habitaciones Disponibles:")
                    .font(.headline)
                    .foregroundColor(.white)
                
                List(roomManager.habitaciones.filter { $0.dispo }) { habitacion in
                    VStack(alignment: .leading) {
                        Text(habitacion.title)
                            .font(.headline)
                        Text(habitacion.body)
                            .font(.subheadline)
                        Text("Precio: $\(habitacion.price, specifier: "%.2f")")
                            .font(.subheadline)
                    }
                }
            }
        }
    }
}

// MARK: - WebView

struct WebView: UIViewRepresentable {
    let url: URL
    
    func makeUIView(context: Context) -> WKWebView {
        return WKWebView()
    }
    
    func updateUIView(_ uiView: WKWebView, context: Context) {
        let request = URLRequest(url: url)
        uiView.load(request)
    }
}

struct BaseView: View {
    let url = URL(string: "https://app.powerbi.com/view?r=eyJrIjoiNmY0MGM4MDItZDYyOS00Y2Q5LTg2M2EtNzdhM2NlZDA5ZmNhIiwidCI6Ijc4MjQ5NGRkLWEwZGUtNDk2Zi1iOTg1LTYyM2NiMjFhZmU3MSJ9")!
    
    var body: some View {
        WebView(url: url)
    }
}

// MARK: - App Principal

@main
struct MiAplicacionApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView(roomManager: RoomManager.shared)
        }
    }
}
```
### **RESPUESTA** 
## <CODE>
### HTMLS\CSS\JS
#### index.html
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
#### styles.css
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
#### script.js
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

### Imagenes como se 
## ve
# despues

<div style="display: flex; gap: 10px;">
  <img src="https://raw.githubusercontent.com/Jordanbuco/Drive/main/Markdown/0/imgs/IMG_0513.png" alt="Imagen 1 - Miniatura" width="100"/>
  <img src="https://raw.githubusercontent.com/Jordanbuco/Drive/main/Markdown/0/imgs/IMG_0514.png" alt="Imagen 2 - Miniatura" width="100"/>
  <img src="https://raw.githubusercontent.com/Jordanbuco/Drive/main/Markdown/0/imgs/IMG_0515.png" alt="Imagen 1 - Miniatura" width="100"/>
  <img src="https://raw.githubusercontent.com/Jordanbuco/Drive/main/Markdown/0/imgs/IMG_0516.png" alt="Imagen 2 - Miniatura" width="100"/>
  
</div>
