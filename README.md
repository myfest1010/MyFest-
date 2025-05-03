# MyFest 
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Eventos Noturnos</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        #map {
            height: 100vh;
            width: 100%;
        }
        .sidebar {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 999;
            background: rgba(255,255,255,0.9);
            padding: 15px;
            border-radius: 10px;
        }
        .filter-group {
            margin-bottom: 10px;
        }
        .marker-icon {
            width: 24px;
            height: 24px;
            background-size: cover;
            border-radius: 50%;
        }
    </style>
</head>
<body>
    <div class="sidebar">
        <h5>Filtrar Eventos</h5>
        <div class="filter-group">
            <label><input type="checkbox" class="filter" value="música"> Música</label>
            <label><input type="checkbox" class="filter" value="festa"> Festa</label>
            <label><input type="checkbox" class="filter" value="jantar"> Jantar</label>
        </div>
        <div class="filter-group">
            <label><input type="checkbox" class="price-filter" value="gratuito"> Gratuito</label>
            <label><input type="checkbox" class="price-filter" value="pagamento"> Pago</label>
        </div>
        <button class="btn btn-primary btn-sm w-100" onclick="filterEvents()">Aplicar Filtros</button>
        <hr>
        <button class="btn btn-success btn-sm w-100" data-bs-toggle="modal" data-bs-target="#loginModal">Entrar</button>
        <button class="btn btn-secondary btn-sm w-100 mt-2" data-bs-toggle="modal" data-bs-target="#registerModal">Cadastrar Estabelecimento</button>
    </div>
    <div id="map"></div>

    <!-- Login Modal -->
    <div class="modal fade" id="loginModal" tabindex="-1" aria-labelledby="loginModalLabel" aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="loginModalLabel">Login</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <form id="loginForm">
                        <div class="mb-3">
                            <label class="form-label">Email</label>
                            <input type="email" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Senha</label>
                            <input type="password" class="form-control" required>
                        </div>
                        <button type="submit" class="btn btn-primary">Entrar</button>
                    </form>
                </div>
            </div>
        </div>
    </div>

    <!-- Register Modal -->
    <div class="modal fade" id="registerModal" tabindex="-1" aria-labelledby="registerModalLabel" aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="registerModalLabel">Cadastrar Estabelecimento</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <form id="registerForm">
                        <div class="mb-3">
                            <label class="form-label">Nome do Estabelecimento</label>
                            <input type="text" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Categoria</label>
                            <select class="form-select" required>
                                <option value="música">Música</option>
                                <option value="festa">Festa</option>
                                <option value="jantar">Jantar</option>
                            </select>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Endereço</label>
                            <input type="text" class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Descrição</label>
                            <textarea class="form-control" rows="3" required></textarea>
                        </div>
                        <button type="submit" class="btn btn-success">Cadastrar</button>
                    </form>
                </div>
            </div>
        </div>
    </div>

    <!-- Checkout Modal -->
    <div class="modal fade" id="checkoutModal" tabindex="-1" aria-labelledby="checkoutModalLabel" aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="checkoutModalLabel">Comprar Ingresso</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <p id="eventTitle"></p>
                    <form id="checkoutForm">
                        <div class="mb-3">
                            <label class="form-label">Quantidade</label>
                            <input type="number" class="form-control" min="1" value="1" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Pagamento</label>
                            <select class="form-select" required>
                                <option value="cartão">Cartão de Crédito</option>
                                <option value="pix">Pix</option>
                            </select>
                        </div>
                        <button type="submit" class="btn btn-primary">Finalizar Compra</button>
                    </form>
                </div>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/js/bootstrap.bundle.min.js"></script>
    <script>
        let map;
        let markers = [];
        let currentLocationMarker;

        const eventos = [
            {
                id: 1,
                nome: "Bar da Noite",
                categoria: "música",
                preco: "pagamento",
                lat: -23.550520,
                lng: -46.633308,
                descricao: "Música ao vivo com DJ residente",
                imagem: "https://maps.google.com/mapfiles/ms/icons/blue-dot.png"
            },
            {
                id: 2,
                nome: "Festa na Praia",
                categoria: "festa",
                preco: "gratuito",
                lat: -23.558670,
                lng: -46.659830,
                descricao: "Festa open bar na areia",
                imagem: "https://maps.google.com/mapfiles/ms/icons/red-dot.png"
            },
            {
                id: 3,
                nome: "Jantar Romântico",
                categoria: "jantar",
                preco: "pagamento",
                lat: -23.561210,
                lng: -46.630450,
                descricao: "Jantar com vista para a cidade",
                imagem: "https://maps.google.com/mapfiles/ms/icons/green-dot.png"
            }
        ];

        function initMap() {
            map = new google.maps.Map(document.getElementById("map"), {
                center: { lat: -23.550520, lng: -46.633308 },
                zoom: 14
            });

            // Adicionar marcadores iniciais
            eventos.forEach(evento => {
                const marker = new google.maps.Marker({
                    position: { lat: evento.lat, lng: evento.lng },
                    map: map,
                    title: evento.nome,
                    icon: evento.imagem,
                    categoria: evento.categoria,
                    preco: evento.preco
                });

                const infoWindow = new google.maps.InfoWindow({
                    content: `<strong>${evento.nome}</strong><br>${evento.descricao}<br><button class="btn btn-sm btn-primary mt-2" onclick="openCheckoutModal('${evento.nome}')">Comprar Ingresso</button>`
                });

                marker.addListener("click", () => {
                    infoWindow.open(map, marker);
                });

                markers.push(marker);
            });

            // Obter localização do usuário
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(position => {
                    const userLocation = {
                        lat: position.coords.latitude,
                        lng: position.coords.longitude
                    };
                    
                    map.setCenter(userLocation);
                    
                    if (currentLocationMarker) currentLocationMarker.setMap(null);
                    
                    currentLocationMarker = new google.maps.Marker({
                        position: userLocation,
                        map: map,
                        title: "Você está aqui",
                        icon: {
                            path: google.maps.SymbolPath.CIRCLE,
                            scale: 8,
                            fillColor: "#FF0000",
                            fillOpacity: 1,
                            strokeWeight: 2,
                            strokeColor: "#FFFFFF"
                        }
                    });
                });
            }
        }

        function filterEvents() {
            const selectedCategories = Array.from(document.querySelectorAll('.filter:checked')).map(cb => cb.value);
            const selectedPrices = Array.from(document.querySelectorAll('.price-filter:checked')).map(cb => cb.value);
            
            markers.forEach(marker => {
                const categoriaMatch = selectedCategories.length === 0 || selectedCategories.includes(marker.categoria);
                const precoMatch = selectedPrices.length === 0 || selectedPrices.includes(marker.preco);
                
                marker.setVisible(categoriaMatch && precoMatch);
            });
        }

        function openCheckoutModal(eventName) {
            document.getElementById('eventTitle').innerText = `Comprar ingresso para: ${eventName}`;
            new bootstrap.Modal(document.getElementById('checkoutModal')).show();
        }

        // Handlers de formulários
        document.getElementById('loginForm').addEventListener('submit', function(e) {
            e.preventDefault();
            alert('Login simulado com sucesso!');
            bootstrap.Modal.getInstance(document.getElementById('loginModal')).hide();
        });

        document.getElementById('registerForm').addEventListener('submit', function(e) {
            e.preventDefault();
            alert('Estabelecimento cadastrado com sucesso!');
            bootstrap.Modal.getInstance(document.getElementById('registerModal')).hide();
        });

        document.getElementById('checkoutForm').addEventListener('submit', function(e) {
            e.preventDefault();
            alert('Compra realizada com sucesso! (Simulação)');
            bootstrap.Modal.getInstance(document.getElementById('checkoutModal')).hide();
        });
    </script>
    <script async defer 
        src="https://maps.googleapis.com/maps/api/js?key=AIzaSyBOsNF4QN1QYChxmL6deFqDfSdb9E-oDJk&callback=initMap">
    </script>
</body>
</html>
