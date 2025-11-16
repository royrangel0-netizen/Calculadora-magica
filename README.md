<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculadora Mágica de Inventario</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    <script src="https://unpkg.com/@phosphor-icons/web"></script>
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.8.2/jspdf-autotable.umd.min.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
        }
        /* Estilos para los botones de acción de la tabla */
        .action-btn {
            @apply p-1.5 rounded-md shadow-sm transition-all duration-150;
        }
        .edit-btn {
            @apply bg-blue-100 text-blue-700 hover:bg-blue-200;
        }
        .delete-btn {
            @apply bg-red-100 text-red-700 hover:bg-red-200;
        }
         /* Mensaje de carga */
        #loading-message {
            @apply fixed inset-0 bg-white/80 backdrop-blur-sm flex items-center justify-center text-xl font-semibold text-gray-700 z-50;
        }
        /* Ocultar spinners en campos de número */
        input[type=number]::-webkit-inner-spin-button,
        input[type=number]::-webkit-outer-spin-button {
            -webkit-appearance: none;
            margin: 0;
        }
        input[type=number] {
            -moz-appearance: textfield;
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800">

    <div class="container mx-auto p-4 md:p-6 lg:p-8 max-w-7xl">

        <header class="mb-6">
            <h1 id="main-title" class="text-3xl font-bold text-gray-800">Calculadora Mágica</h1>
            <p class="text-gray-600">Gestor de inventario y precios en tiempo real.</p>
            <div class="mt-2 text-xs text-gray-500">
                <span class="font-semibold">ID de Sesión:</span> <span id="userIdDisplay">Cargando...</span>
            </div>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">

            <div class="lg:col-span-1 flex flex-col gap-6">

                <div class="bg-white p-5 rounded-lg shadow-sm border border-gray-200">
                    <h2 class="text-lg font-semibold mb-3 border-b pb-2">Respaldo y Restauración</h2>
                    <div class="grid grid-cols-2 gap-3">
                        <button id="backupBtn" class="bg-green-600 text-white px-4 py-2 rounded-md hover:bg-green-700 transition-colors flex items-center justify-center gap-2">
                            <i class="ph ph-download-simple"></i>
                            Descargar
                        </button>
                        <button id="restoreBtn" class="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700 transition-colors flex items-center justify-center gap-2">
                            <i class="ph ph-upload-simple"></i>
                            Restaurar
                        </button>
                        <input type="file" id="restoreFileInput" class="hidden" accept=".json">
                    </div>
                </div>

                <div class="bg-white p-5 rounded-lg shadow-sm border border-gray-200">
                    <h2 class="text-lg font-semibold mb-3 border-b pb-2">Configuración</h2>
                    
                    <div class="mb-4">
                        <label for="establishmentName" class="block text-sm font-medium text-gray-600 mb-1">Nombre del Establecimiento</label>
                        <div class="flex gap-2">
                            <input type="text" id="establishmentName" class="flex-grow w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500">
                            <button id="saveNameBtn" class="bg-gray-700 text-white px-4 py-2 rounded-md hover:bg-gray-800 transition-colors">
                                Guardar
                            </button>
                        </div>
                    </div>

                    <div>
                        <label for="bcvRate" class="block text-sm font-medium text-gray-600 mb-1">Tasa BCV</label>
                        <div class="flex gap-2">
                            <input type="number" id="bcvRate" placeholder="Ej: 36.50" class="flex-grow w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500">
                            <button id="updateBcvBtn" class="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700 transition-colors">
                                Actualizar
                            </button>
                        </div>
                    </div>
                </div>

                <div class="bg-white p-5 rounded-lg shadow-sm border border-gray-200 sticky top-6">
                    <h2 id="productFormTitle" class="text-lg font-semibold mb-3 border-b pb-2">Agregar Producto</h2>
                    <form id="productForm" class="flex flex-col gap-4">
                        <input type="hidden" id="editingProductId">
                        
                        <div>
                            <label for="productName" class="block text-sm font-medium text-gray-600 mb-1">Nombre del Producto</label>
                            <input type="text" id="productName" class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500" required>
                        </div>

                        <div class="grid grid-cols-2 gap-4">
                            <div>
                                <label for="productCostUSD" class="block text-sm font-medium text-gray-600 mb-1">Costo ($)</label>
                                <input type="number" id="productCostUSD" step="0.01" placeholder="Ej: 10.50" class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500" required>
                            </div>
                            <div>
                                <label for="productProfitPercentage" class="block text-sm font-medium text-gray-600 mb-1">Ganancia (%)</label>
                                <input type="number" id="productProfitPercentage" step="1" placeholder="Ej: 30" class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500" required>
                            </div>
                        </div>

                        <div>
                            <label for="productStock" class="block text-sm font-medium text-gray-600 mb-1">Existencias</label>
                            <input type="number" id="productStock" step="1" placeholder="Ej: 50" class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500" required>
                        </div>

                        <div class="flex gap-3 mt-2">
                            <button type="submit" id="saveProductBtn" class="flex-1 bg-green-600 text-white px-4 py-2 rounded-md hover:bg-green-700 transition-colors">
                                Guardar Producto
                            </button>
                            <button type="button" id="cancelEditBtn" class="flex-1 bg-gray-200 text-gray-700 px-4 py-2 rounded-md hover:bg-gray-300 transition-colors" style="display: none;">
                                Cancelar
                            </button>
                        </div>
                    </form>
                </div>

            </div>

            <div class="lg:col-span-2 bg-white p-5 rounded-lg shadow-sm border border-gray-200">
                
                <div class="flex justify-between items-center mb-3">
                    <h2 class="text-lg font-semibold">Inventario de Productos</h2>
                    <button id="downloadPdfBtn" class="bg-red-600 text-white px-3 py-1.5 rounded-md hover:bg-red-700 transition-colors flex items-center justify-center gap-1.5 text-sm shadow-sm">
                        <i class="ph ph-file-pdf"></i>
                        Descargar PDF
                    </button>
                </div>
                
                <div class="mb-4">
                    <label for="searchInput" class="sr-only">Buscar producto</label>
                    <div class="relative">
                        <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                            <i class="ph ph-magnifying-glass text-gray-400"></i>
                        </div>
                        <input type="text" id="searchInput" placeholder="Buscar producto por nombre..." class="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500">
                    </div>
                </div>

                <div class="overflow-x-auto">
                    <table class_all="w-full min-w-[700px] text-sm text-left">
                        <thead class="bg-gray-50 text-gray-600 uppercase text-xs">
                            <tr>
                                <th class="px-4 py-3">Producto</th>
                                <th class="px-4 py-3">Costo ($)</th>
                                <th class="px-4 py-3">Ganancia</th>
                                <th class="px-4 py-3">Venta ($)</th>
                                <th class="px-4 py-3">Venta (Bs)</th>
                                <th class="px-4 py-3">Existencia</th>
                                <th class="px-4 py-3">Acciones</th>
                            </tr>
                        </thead>
                        <tbody id="inventoryTableBody" class="divide-y divide-gray-200">
                            <tr>
                                <td colspan="7" class="px-4 py-6 text-center text-gray-500">Cargando productos...</td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="mt-4 text-sm text-gray-600 border-t pt-3">
                    Tasa BCV actual: <span id="currentBcvRate" class="font-semibold">N/A</span>
                </div>
            </div>

        </div>
    </div>

    <div id="loading-message">
        <i class="ph ph-circle-notch animate-spin text-4xl mr-3"></i>
        Conectando a la base de datos...
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { 
            getAuth, 
            signInAnonymously, 
            signInWithCustomToken, 
            onAuthStateChanged 
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { 
            getFirestore, 
            doc, 
            getDoc, 
            setDoc, 
            updateDoc, 
            deleteDoc, 
            onSnapshot, 
            collection, 
            query,
            addDoc,
            getDocs,
            writeBatch,
            setLogLevel
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- VARIABLES GLOBALES ---
        let db, auth, userId, appId;
        let settingsRef, productsRef;
        let globalBcvRate = 0;
        let allProducts = []; // Caché local para búsqueda
        
        // Elementos del DOM
        const loadingMessage = document.getElementById('loading-message');
        const userIdDisplay = document.getElementById('userIdDisplay');
        const mainTitle = document.getElementById('main-title');
        
        // Config
        const establishmentNameEl = document.getElementById('establishmentName');
        const saveNameBtn = document.getElementById('saveNameBtn');
        const bcvRateEl = document.getElementById('bcvRate');
        const updateBcvBtn = document.getElementById('updateBcvBtn');
        const currentBcvRateEl = document.getElementById('currentBcvRate');
        
        // Formulario de producto
        const productForm = document.getElementById('productForm');
        const productFormTitle = document.getElementById('productFormTitle');
        const editingProductId = document.getElementById('editingProductId');
        const productNameEl = document.getElementById('productName');
        const productCostUSDEl = document.getElementById('productCostUSD');
        const productProfitPercentageEl = document.getElementById('productProfitPercentage');
        const productStockEl = document.getElementById('productStock');
        const saveProductBtn = document.getElementById('saveProductBtn');
        const cancelEditBtn = document.getElementById('cancelEditBtn');

        // Inventario
        const searchInput = document.getElementById('searchInput');
        const inventoryTableBody = document.getElementById('inventoryTableBody');
        const downloadPdfBtn = document.getElementById('downloadPdfBtn'); // NUEVO: Botón PDF

        // Respaldo
        const backupBtn = document.getElementById('backupBtn');
        const restoreBtn = document.getElementById('restoreBtn');
        const restoreFileInput = document.getElementById('restoreFileInput');

        // --- INICIALIZACIÓN DE FIREBASE ---
        
        async function initFirebase() {
            // Usar variables globales de configuración
            const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
            appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

            if (!firebaseConfig.apiKey) {
                console.error("Configuración de Firebase no encontrada.");
                loadingMessage.innerText = "Error: Configuración de Firebase no encontrada.";
                return;
            }

            try {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                setLogLevel('Debug'); // Habilitar logs de Firestore

                // Autenticación
                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        userIdDisplay.textContent = userId;
                        console.log("Usuario autenticado:", userId);
                        await loadData(); // Cargar datos después de la autenticación
                    } else {
                        // Intentar con token o anónimo
                        const token = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
                        if (token) {
                            await signInWithCustomToken(auth, token);
                        } else {
                            await signInAnonymously(auth);
                        }
                    }
                });
            } catch (error) {
                console.error("Error al inicializar Firebase:", error);
                loadingMessage.innerText = "Error al conectar con la base de datos.";
            }
        }

        // --- CARGA DE DATOS (ONSNAPSHOT) ---

        async function loadData() {
            if (!userId) return;

            loadingMessage.style.display = 'flex';
            
            // Definir rutas de Firestore
            // Usamos "public/data" para que los datos sean compartidos por cualquier usuario de esta app
            const dataPrefix = `artifacts/${appId}/public/data`;
            settingsRef = doc(db, dataPrefix, "config", "settings");
            productsRef = collection(db, dataPrefix, "products");

            // 1. Suscribirse a cambios en la Configuración (Tasa y Nombre)
            try {
                onSnapshot(settingsRef, (docSnap) => {
                    if (docSnap.exists()) {
                        const settings = docSnap.data();
                        globalBcvRate = parseFloat(settings.bcvRate) || 0;
                        
                        // Actualizar UI de configuración
                        establishmentNameEl.value = settings.establishmentName || '';
                        bcvRateEl.value = globalBcvRate;
                        mainTitle.textContent = settings.establishmentName || 'Calculadora Mágica';
                        
                        // Actualizar UI de inventario
                        currentBcvRateEl.textContent = formatCurrency(globalBcvRate, 'Bs');
                        
                        // Re-renderizar tabla para actualizar precios en Bs
                        renderInventoryTable();
                    } else {
                        console.log("No hay documento de configuración. Creando uno...");
                        setDoc(settingsRef, { bcvRate: 0, establishmentName: "Calculadora Mágica" });
                    }
                });
            } catch (e) {
                console.error("Error al cargar configuración:", e);
            }

            // 2. Suscribirse a cambios en los Productos
            try {
                onSnapshot(query(productsRef), (snapshot) => {
                    allProducts = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                    renderInventoryTable(); // Renderizar con todos los productos
                    
                    if (loadingMessage.style.display !== 'none') {
                        loadingMessage.style.display = 'none';
                    }
                });
            } catch (e) {
                console.error("Error al cargar productos:", e);
            }
        }

        // --- RENDERIZADO DE LA TABLA ---

        function renderInventoryTable() {
            if (!inventoryTableBody) return;

            const filter = searchInput.value.toLowerCase();
            const filteredProducts = allProducts.filter(p => 
                p.name && p.name.toLowerCase().includes(filter)
            );

            inventoryTableBody.innerHTML = ''; // Limpiar tabla

            if (filteredProducts.length === 0) {
                inventoryTableBody.innerHTML = `<tr><td colspan="7" class="px-4 py-6 text-center text-gray-500">No se encontraron productos.</td></tr>`;
                return;
            }
            
            // Ordenar alfabéticamente
            filteredProducts.sort((a, b) => a.name.localeCompare(b.name));

            filteredProducts.forEach(product => {
                const costUSD = parseFloat(product.costUSD) || 0;
                const profitPercentage = parseFloat(product.profitPercentage) || 0;
                const stock = parseInt(product.stock) || 0;

                const salePriceUSD = costUSD * (1 + profitPercentage / 100);
                const salePriceBs = salePriceUSD * globalBcvRate;

                const tr = document.createElement('tr');
                tr.className = "hover:bg-gray-50";
                tr.innerHTML = `
                    <td class="px-4 py-3 font-medium">${product.name}</td>
                    <td class="px-4 py-3">${formatCurrency(costUSD, '$')}</td>
                    <td class="px-4 py-3">${profitPercentage.toFixed(0)}%</td>
                    <td class="px-4 py-3 font-semibold">${formatCurrency(salePriceUSD, '$')}</td>
                    <td class="px-4 py-3 font-semibold">${formatCurrency(salePriceBs, 'Bs')}</td>
                    <td class="px-4 py-3">${stock}</td>
                    <td class="px-4 py-3">
                        <div class="flex gap-2">
                            <button class="action-btn edit-btn" data-id="${pr# Calculadora-magica
Crea una calculadora con inventario
