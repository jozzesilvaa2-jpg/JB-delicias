<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Lanches da B - JB DELICIAS</title>
    <!-- Carrega Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        'primary-red': '#D70000', // Vermelho vibrante
                        'dark-bg': '#1a1a1a', // Fundo principal escuro
                        'highlight-yellow': '#FFD700', // Amarelo para destaque
                        'whatsapp-green': '#25D366', // Verde do WhatsApp
                    },
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    }
                }
            }
        }
    </script>
    <!-- Estilo customizado para o layout mobile -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #000000; /* Fundo preto */
        }
        /* Garantindo que o corpo principal fique abaixo do header fixo */
        #main-content {
            padding-top: 130px; /* Ajuste para acomodar o header e a barra de navegação */
            padding-bottom: 20px;
        }
        /* Estiliza a barra de rolagem (opcional, para estética) */
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-thumb {
            background: #555;
            border-radius: 3px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #D70000;
        }
        .text-break-all {
             word-break: break-all;
        }
    </style>
</head>
<body class="text-white">

    <!-- Lógica JS (100% Local) -->
    <script>
        // --- CONSTANTES E ESTADO INICIAL ---
        
        // Configuração de Frete
        const FREE_SHIPPING_THRESHOLD = 90.00; // Frete grátis a partir de R$ 90,00
        const SHIPPING_FEE = 10.00; // Taxa de entrega padrão
        
        // Credenciais hardcoded para o administrador (apenas para acesso ao editor)
        const ADMIN_USER = 'JB delicias';
        const ADMIN_PASS = '304050';

        let cart = []; // A sacola será carregada do localStorage
        let currentView = 'menu'; // 'menu', 'invite', 'cart'
        let isAdmin = false;
        let nextCartItemId = 1;

        // Dados estáticos de exemplo (Cardápio)
        let menuItems = [
            { id: 'xbacon', name: 'Xis Bacon da B', description: 'O Original! Pão, carne, queijo, bacon, alface e tomate.', price: 29.90, img: 'https://placehold.co/100x100/3B0000/FFFFFF?text=Xis' },
            { id: 'esp3', name: 'Especial 3 (Combo)', description: 'Hambúrguer + Batata Média + Coca 220ml.', price: 29.90, img: 'https://placehold.co/100x100/003B00/FFFFFF?text=Combo' },
            { id: 'esp2', name: 'Especial 2 (Bacon)', description: 'Bacon Burger + Tropical + Coca 220ml.', price: 26.90, img: 'https://placehold.co/100x100/3B3B00/FFFFFF?text=Bacon' },
            { id: 'esp1', name: 'Especial 1 (Frango)', description: 'X-Burger + Frango Salada + Coca 220ml.', price: 24.90, img: 'https://placehold.co/100x100/00003B/FFFFFF?text=Frango' },
            { id: 'refri', name: 'Refrigerante 2L', description: 'Coca-Cola ou Guaraná.', price: 12.00, img: 'https://placehold.co/100x100/333333/FFFFFF?text=Refri' },
        ];
        
        // Dados de Configuração Gerais (Salvos localmente)
        let appSettings = {
            rewardText: 'a cada 3 amigos que comprarem você ganha 70% de desconto e 1 refrigerante 600 ML grátis.',
            whatsappNumber: '555181114714', // Padrão: 51 81114714 com código do país (55)
            qrCodeImage: 'https://placehold.co/200x200/555555/FFFFFF?text=QR+Code+Padrão' // URL de Imagem/Base64 para o QR Code
        };


        // --- FUNÇÕES DE ARMAZENAMENTO LOCAL ---

        /** Carrega o estado da aplicação (sacola e configurações) do LocalStorage ao iniciar */
        function loadStateFromStorage() {
            try {
                // Carregar Sacola
                const storedCart = localStorage.getItem('lanches_da_b_cart');
                if (storedCart) {
                    cart = JSON.parse(storedCart);
                    const maxId = cart.reduce((max, item) => Math.max(max, item.id), 0);
                    nextCartItemId = maxId + 1;
                }
                
                // Carregar Configurações
                const storedSettings = localStorage.getItem('lanches_da_b_settings');
                if (storedSettings) {
                    // Mescla as configurações carregadas com as padrão para garantir que novas chaves existam
                    appSettings = { ...appSettings, ...JSON.parse(storedSettings) };
                }

            } catch (e) {
                console.error("Erro ao carregar o estado do localStorage:", e);
                cart = [];
            }
        }

        /** Salva a sacola e as configurações no LocalStorage após cada alteração */
        function saveStateToStorage() {
            try {
                localStorage.setItem('lanches_da_b_cart', JSON.stringify(cart));
                localStorage.setItem('lanches_da_b_settings', JSON.stringify(appSettings));
            } catch (e) {
                console.error("Erro ao salvar o estado no localStorage:", e);
            }
        }

        // --- FUNÇÕES DE ADMINISTRAÇÃO ---

        /** Tenta logar o administrador com credenciais hardcoded */
        function handleLogin(event) {
            event.preventDefault();
            const username = document.getElementById('admin-user').value;
            const password = document.getElementById('admin-pass').value;
            const errorElement = document.getElementById('login-error');
            errorElement.textContent = '';

            if (username === ADMIN_USER && password === ADMIN_PASS) {
                isAdmin = true;
                hideModal('admin-modal');
                // Abre o editor de cardápio
                showModal('menu-editor-modal');
                setAdminEditorView('menu');
            } else {
                errorElement.textContent = 'Usuário ou senha incorretos.';
            }
        }
        
        /** Abre o painel do administrador (login ou editor) */
        function openAdminPanel() {
            if (isAdmin) {
                showModal('menu-editor-modal');
                setAdminEditorView('menu');
            } else {
                showModal('admin-modal');
                document.getElementById('login-error').textContent = '';
                document.getElementById('admin-user').value = '';
                document.getElementById('admin-pass').value = '';
            }
        }

        /** Controla qual seção do modal Admin está visível */
        function setAdminEditorView(view) {
            document.getElementById('item-list-section').classList.add('hidden');
            document.getElementById('edit-item-section').classList.add('hidden');
            document.getElementById('settings-editor-section').classList.add('hidden');

            document.querySelectorAll('.admin-tab-button').forEach(btn => btn.classList.remove('bg-primary-red', 'text-white'));

            switch (view) {
                case 'menu':
                    document.getElementById('item-list-section').classList.remove('hidden');
                    document.getElementById('admin-menu-tab').classList.add('bg-primary-red', 'text-white');
                    renderMenuEditor();
                    break;
                case 'settings':
                    document.getElementById('settings-editor-section').classList.remove('hidden');
                    document.getElementById('admin-settings-tab').classList.add('bg-primary-red', 'text-white');
                    renderSettingsEditor();
                    break;
                case 'edit-item':
                    document.getElementById('edit-item-section').classList.remove('hidden');
                    document.getElementById('admin-menu-tab').classList.add('bg-primary-red', 'text-white'); 
                    break;
            }
        }

        /** Prepara o formulário para adicionar ou editar um item */
        function openEditForm(itemId = null) {
            setAdminEditorView('edit-item');
            const formTitle = document.getElementById('edit-form-title');
            const form = document.getElementById('menu-edit-form');
            form.reset();
            form.dataset.itemId = '';

            if (itemId) {
                const item = menuItems.find(i => i.id === itemId);
                if (!item) return;
                
                formTitle.textContent = `Editar: ${item.name}`;
                document.getElementById('edit-name').value = item.name;
                document.getElementById('edit-description').value = item.description;
                document.getElementById('edit-price').value = item.price.toFixed(2);
                document.getElementById('edit-img').value = item.img;
                form.dataset.itemId = itemId;
            } else {
                formTitle.textContent = 'Adicionar Novo Item';
            }
        }

        /** Salva um novo item ou atualiza um existente (SALVA APENAS NO ESTADO LOCAL) */
        function saveMenuItem(event) {
            event.preventDefault();

            const form = document.getElementById('menu-edit-form');
            const itemId = form.dataset.itemId;
            
            const name = document.getElementById('edit-name').value;
            const description = document.getElementById('edit-description').value;
            const price = parseFloat(document.getElementById('edit-price').value);
            const img = document.getElementById('edit-img').value;

            if (!name || !description || isNaN(price) || price <= 0) {
                showModal('info-modal', 'Campos Inválidos', 'Nome, descrição e preço válido são obrigatórios.');
                return;
            }

            const itemData = { name, description, price: price, img };

            if (itemId) {
                const index = menuItems.findIndex(i => i.id === itemId);
                if (index !== -1) {
                    menuItems[index] = { ...menuItems[index], ...itemData };
                    showModal('info-modal', 'Sucesso!', `Item '${name}' atualizado no estado local.`);
                }
            } else {
                // Cria um ID único local
                const newId = (Math.random() + 1).toString(36).substring(2);
                menuItems.push({ ...itemData, id: newId });
                showModal('info-modal', 'Sucesso!', `Novo item '${name}' adicionado ao cardápio local.`);
            }
            
            setAdminEditorView('menu'); // Volta para a lista
            if (currentView === 'menu') renderMenu(); // Atualiza a view do cliente
        }
        
        /** Salva as configurações gerais (SALVA APENAS NO ESTADO LOCAL) */
        function saveSettings(event) {
            event.preventDefault();

            const rewardText = document.getElementById('edit-invite-text').value;
            // Remove caracteres não-numéricos do número de WhatsApp
            const whatsappNumber = document.getElementById('edit-whatsapp-number').value.replace(/[^0-9]/g, ''); 
            // O valor aqui pode ser uma URL ou uma string Base64
            const qrCodeImage = document.getElementById('edit-qr-code-img').value;

            if (!rewardText || !whatsappNumber) {
                showModal('info-modal', 'Campos Inválidos', 'O texto de recompensa e o número de WhatsApp são obrigatórios.');
                return;
            }
            
            appSettings.rewardText = rewardText;
            appSettings.whatsappNumber = whatsappNumber;
            appSettings.qrCodeImage = qrCodeImage;
            
            saveStateToStorage();
            
            showModal('info-modal', 'Sucesso!', 'Configurações salvas no estado local.');
            if (currentView === 'invite') renderInvite(); // Atualiza a view do cliente
        }

        /** Deleta um item do cardápio (DELETA APENAS NO ESTADO LOCAL) */
        function deleteMenuItem(itemId) {
            const item = menuItems.find(i => i.id === itemId);
            if (!item) return;

            if (!window.confirm(`Tem certeza que deseja EXCLUIR o item: ${item.name}? Esta mudança é temporária e não será salva.`)) return;

            menuItems = menuItems.filter(i => i.id !== itemId);
            showModal('info-modal', 'Sucesso!', `Item '${item.name}' excluído do estado local.`);
            
            setAdminEditorView('menu'); // Atualiza a lista de edição
            if (currentView === 'menu') renderMenu(); // Atualiza a view do cliente
        }

        /** Volta da edição para a lista de itens no admin (apenas oculta a seção de edição de item) */
        function cancelEdit() {
            setAdminEditorView('menu');
        }

        // --- FUNÇÕES DE MANIPULAÇÃO DA SACULA ---

        /** Adiciona um item à sacola local */
        function addToCart(itemId) {
            const item = menuItems.find(i => i.id === itemId);
            if (!item) return;

            const itemInCart = cart.find(i => i.menuItemId === itemId);

            if (itemInCart) {
                itemInCart.quantity += 1;
            } else {
                cart.push({
                    id: nextCartItemId++, // ID único no carrinho
                    menuItemId: itemId,
                    name: item.name,
                    price: item.price,
                    quantity: 1,
                });
            }
            saveStateToStorage();
            updateCartBadge();
        }

        /** Altera a quantidade de um item na sacola local */
        function updateCartItemQuantity(cartItemId, delta) {
            const itemIndex = cart.findIndex(i => i.id === cartItemId);
            if (itemIndex === -1) return;

            const newQuantity = cart[itemIndex].quantity + delta;

            if (newQuantity <= 0) {
                cart = cart.filter(i => i.id !== cartItemId);
            } else {
                cart[itemIndex].quantity = newQuantity;
            }
            
            saveStateToStorage();
            updateCartBadge();
            if (currentView === 'cart') renderCart();
        }

        /** Limpa a sacola local */
        function clearCart() {
             if (cart.length === 0) return;
             cart = [];
             saveStateToStorage();
             updateCartBadge();
             if (currentView === 'cart') renderCart();
             showModal('info-modal', 'Sacola Limpa', 'Todos os itens foram removidos do seu carrinho.');
        }
        
        /** Gera a mensagem e abre o WhatsApp */
        function finalizeOrder() {
            if (cart.length === 0) {
                showModal('info-modal', 'Carrinho Vazio', 'Adicione itens ao carrinho antes de finalizar o pedido.');
                return;
            }

            const whatsappNumber = appSettings.whatsappNumber;
            
            let itemsTotal = 0;
            let message = "Olá! Gostaria de fazer o seguinte pedido do Lanches da B:\n\n";

            cart.forEach((item) => {
                const itemPrice = parseFloat(item.price) || 0;
                const subtotal = itemPrice * item.quantity;
                itemsTotal += subtotal;
                message += `${item.quantity}x ${item.name} - R$ ${subtotal.toFixed(2).replace('.', ',')}\n`;
            });

            const shippingFee = itemsTotal >= FREE_SHIPPING_THRESHOLD ? 0 : SHIPPING_FEE;
            const finalTotal = itemsTotal + shippingFee;
            
            const shippingText = shippingFee === 0 ? "GRÁTIS (Pedido acima de R$ 90,00)" : `R$ ${shippingFee.toFixed(2).replace('.', ',')}`;

            message += `\n--- Resumo ---\n`;
            message += `Subtotal dos Itens: R$ ${itemsTotal.toFixed(2).replace('.', ',')}\n`;
            message += `Taxa de Entrega: ${shippingText}\n`;
            message += `\n*TOTAL FINAL: R$ ${finalTotal.toFixed(2).replace('.', ',')}*`;
            message += `\n\nPor favor, me informe o endereço de entrega e a forma de pagamento.`; 

            const whatsappUrl = `https://api.whatsapp.com/send?phone=${whatsappNumber}&text=${encodeURIComponent(message)}`;

            // Abre em uma nova aba, garantindo o redirecionamento
            window.open(whatsappUrl, '_blank');
            showModal('info-modal', 'Redirecionando...', 'Você será redirecionado para o WhatsApp para confirmar seu pedido.');
        }

        /** Abre o WhatsApp para falar com o dono, sem mensagem pré-definida */
        function talkToOwner() {
            const message = "Olá, Lanches da B! Tenho uma dúvida ou preciso de ajuda.";
            const whatsappNumber = appSettings.whatsappNumber;
            const whatsappUrl = `https://api.whatsapp.com/send?phone=${whatsappNumber}&text=${encodeURIComponent(message)}`;
            window.open(whatsappUrl, '_blank');
        }

        // --- FUNÇÕES DE RENDERIZAÇÃO E UI ---

        /** Renderiza a contagem de itens na badge da sacola */
        function updateCartBadge() {
            const totalItems = cart.reduce((sum, item) => sum + item.quantity, 0);
            const badge = document.getElementById('cart-badge');
            badge.textContent = totalItems;
            badge.classList.toggle('hidden', totalItems === 0);
        }

        /** Alterna a visualização principal */
        function setView(view) {
            currentView = view;
            const menuContent = document.getElementById('menu-content');
            const inviteContent = document.getElementById('invite-content'); 
            const cartContent = document.getElementById('cart-content');
            const viewTitle = document.getElementById('view-title');

            document.querySelectorAll('.tab-button').forEach(btn => btn.classList.remove('border-b-4', 'border-primary-red', 'text-white'));

            menuContent.classList.add('hidden');
            inviteContent.classList.add('hidden'); 
            cartContent.classList.add('hidden');

            switch (view) {
                case 'menu':
                    menuContent.classList.remove('hidden');
                    viewTitle.textContent = 'Cardápio';
                    document.getElementById('menu-tab').classList.add('border-b-4', 'border-primary-red', 'text-white');
                    renderMenu();
                    break;
                case 'invite': 
                    inviteContent.classList.remove('hidden');
                    viewTitle.textContent = 'Convide Amigos';
                    document.getElementById('invite-tab').classList.add('border-b-4', 'border-primary-red', 'text-white');
                    renderInvite();
                    break;
                case 'cart':
                    cartContent.classList.remove('hidden');
                    viewTitle.textContent = 'Carrinho';
                    document.getElementById('cart-tab').classList.add('border-b-4', 'border-primary-red', 'text-white');
                    renderCart();
                    break;
            }
        }

        /** Renderiza a lista de itens do cardápio */
        function renderMenu() {
            const container = document.getElementById('menu-list');
            container.innerHTML = ''; 

            const promoSection = `
                <div class="bg-primary-red p-4 rounded-xl mb-6 shadow-xl">
                    <h2 class="text-xl font-extrabold text-white text-center">TÁ NA OFERTA DO DIA</h2>
                    <p class="text-sm text-center mt-1 text-gray-200">Todo dia tem promoção especial!</p>
                </div>
                <h3 class="text-xl font-bold text-white mb-4 mt-6">Lanches Especiais</h3>
            `;
            container.innerHTML += promoSection;

            menuItems.forEach(item => {
                const displayPrice = (item.price || 0).toFixed(2).replace('.', ',');

                const itemHTML = `
                    <div class="flex items-center bg-dark-bg p-3 rounded-xl shadow-md mb-3 hover:bg-gray-800 transition duration-150 border-l-4 border-highlight-yellow">
                        <!-- Imagem do item -->
                        <img src="${item.img || 'https://placehold.co/100x100/333333/FFFFFF?text=Item'}" alt="${item.name}" onerror="this.onerror=null;this.src='https://placehold.co/100x100/333333/FFFFFF?text=Item';" class="w-16 h-16 rounded-lg object-cover mr-4">
                        
                        <!-- Detalhes do item -->
                        <div class="flex-grow">
                            <p class="text-base font-semibold text-white">${item.name}</p>
                            <p class="text-xs text-gray-400 line-clamp-2">${item.description}</p>
                        </div>
                        
                        <!-- Preço e Botão Adicionar -->
                        <div class="text-right ml-4 flex-shrink-0">
                            <p class="text-base font-bold text-highlight-yellow">R$ ${displayPrice}</p>
                            <button onclick="window.addToCart('${item.id}')"
                                    class="mt-1 bg-primary-red text-white p-1 rounded-full w-8 h-8 flex items-center justify-center hover:bg-red-600 transition duration-150 shadow-lg">
                                <!-- Ícone de mais -->
                                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                            </button>
                        </div>
                    </div>
                `;
                container.innerHTML += itemHTML;
            });
        }
        
        /** Renderiza o painel de edição do cardápio (Admin) */
        function renderMenuEditor() {
            if (!isAdmin) return;
            const container = document.getElementById('admin-item-list');
            container.innerHTML = '';
            
            container.innerHTML += `
                <button onclick="window.openEditForm()" class="w-full bg-highlight-yellow text-dark-bg font-bold py-3 mb-6 rounded-xl hover:bg-yellow-400 transition duration-150 flex items-center justify-center shadow-lg">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                    Adicionar Novo Item
                </button>
                <h4 class="text-lg font-semibold text-primary-red mb-3">Itens Atuais (${menuItems.length}) (Apenas no Navegador)</h4>
            `;

            menuItems.forEach(item => {
                const itemHTML = `
                    <div class="flex items-center bg-gray-800 p-3 rounded-xl shadow-md mb-3 border border-gray-700">
                        <img src="${item.img || 'https://placehold.co/100x100/333333/FFFFFF?text=Item'}" alt="${item.name}" class="w-10 h-10 rounded-full object-cover mr-3">
                        <div class="flex-grow">
                            <p class="text-sm font-semibold text-white">${item.name}</p>
                            <p class="text-xs text-highlight-yellow">R$ ${parseFloat(item.price).toFixed(2).replace('.', ',')}</p>
                        </div>
                        
                        <!-- Botões de Ação -->
                        <div class="flex space-x-2">
                            <button onclick="window.openEditForm('${item.id}')"
                                    class="text-blue-400 p-1 rounded-full hover:bg-gray-700 transition duration-150">
                                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"></path></svg>
                            </button>
                            <button onclick="window.deleteMenuItem('${item.id}')"
                                    class="text-primary-red p-1 rounded-full hover:bg-gray-700 transition duration-150">
                                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                            </button>
                        </div>
                    </div>
                `;
                container.innerHTML += itemHTML;
            });
        }
        
        /** Renderiza o formulário do Editor de Configurações (Admin) */
        function renderSettingsEditor() {
            if (!isAdmin) return;
            // Configurações de Convite
            document.getElementById('edit-invite-text').value = appSettings.rewardText;
            
            // Configurações de WhatsApp e QR Code
            document.getElementById('edit-whatsapp-number').value = appSettings.whatsappNumber;
            document.getElementById('edit-qr-code-img').value = appSettings.qrCodeImage;
        }

        /** Renderiza o conteúdo da aba "Convide Amigos" */
        function renderInvite() {
            const container = document.getElementById('invite-content');
            const siteUrl = window.location.href; 
            
            // Usa a URL de imagem ou Base64 configurada no admin
            const qrCodeUrl = appSettings.qrCodeImage;

            container.innerHTML = `
                <div class="text-center p-6 rounded-xl bg-dark-bg shadow-2xl border border-highlight-yellow">
                    <h3 class="text-2xl font-black text-primary-red mb-4">Convide Seus Amigos e Ganhe!</h3>
                    <p class="text-white mb-6">Compartilhe o link ou QR Code do nosso cardápio para desbloquear recompensas exclusivas:</p>

                    <!-- QR Code Configurado -->
                    <div class="flex justify-center mb-6">
                        <img src="${qrCodeUrl}" 
                             alt="QR Code para o Site" 
                             onerror="this.onerror=null;this.src='https://placehold.co/200x200/555555/FFFFFF?text=QR+Code+Padrão';" 
                             class="w-48 h-48 rounded-lg border-4 border-white shadow-xl object-contain">
                    </div>

                    <!-- Link do Site -->
                    <p class="text-sm text-gray-400 mb-2">Toque para copiar o Link:</p>
                    <div class="flex justify-center">
                        <p class="text-sm font-mono bg-gray-800 p-2 rounded-lg text-break-all text-highlight-yellow cursor-pointer max-w-full" 
                           onclick="navigator.clipboard.writeText('${siteUrl}').then(() => showModal('info-modal', 'Copiado!', 'Link copiado para a área de transferência.'));">
                            ${siteUrl.length > 30 ? siteUrl.substring(0, 30) + '...' : siteUrl}
                            <span class="ml-2 text-primary-red">
                                <svg class="w-4 h-4 inline-block" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5v2m0-2h2m4 0h2M9 16H6a2 2 0 00-2 2v2a2 2 0 002 2h10a2 2 0 002-2v-2a2 2 0 00-2-2h-3"></path></svg>
                            </span>
                        </p>
                    </div>
                    
                    <!-- Recompensa Editável -->
                    <div class="mt-8 p-4 bg-primary-red rounded-lg shadow-xl border border-highlight-yellow">
                        <p class="text-lg font-bold text-white mb-2">🎁 Sua Recompensa:</p>
                        <p class="text-xl font-extrabold text-highlight-yellow">${appSettings.rewardText}</p>
                    </div>
                    
                    <p class="text-xs text-gray-500 mt-4">A recompensa será aplicada após a confirmação das compras.</p>
                </div>
            `;
        }
        
        /** Renderiza o conteúdo da aba "Carrinho" com cálculo de frete grátis */
        function renderCart() {
            const container = document.getElementById('cart-list');
            const totalSummary = document.getElementById('cart-summary');
            container.innerHTML = '';
            
            let itemsTotal = 0; // Subtotal dos itens
            
            if (cart.length === 0) {
                container.innerHTML = `
                    <div class="text-center p-8 bg-gray-800 rounded-xl mt-4">
                        <p class="text-lg font-semibold text-gray-400">Sua sacola está vazia.</p>
                        <p class="text-sm text-gray-500 mt-2">Adicione itens do nosso cardápio!</p>
                    </div>
                `;
            } else {
                cart.forEach(item => {
                    const itemPrice = parseFloat(item.price) || 0;
                    const subtotal = itemPrice * item.quantity;
                    itemsTotal += subtotal;

                    const itemHTML = `
                        <div class="flex items-center bg-gray-800 p-3 rounded-xl shadow-md border-l-4 border-primary-red mb-3">
                            <!-- Ícone do item -->
                            <div class="w-8 h-8 rounded-full bg-highlight-yellow flex items-center justify-center text-dark-bg font-bold mr-3 text-sm flex-shrink-0">
                                ${item.quantity}x
                            </div>
                            
                            <!-- Detalhes e Preço -->
                            <div class="flex-grow">
                                <p class="text-base font-semibold text-white">${item.name}</p>
                                <p class="text-sm text-gray-400">R$ ${itemPrice.toFixed(2).replace('.', ',')} / un</p>
                            </div>
                            
                            <!-- Controles de Quantidade e Subtotal -->
                            <div class="flex items-center space-x-2 ml-4 flex-shrink-0">
                                <div class="flex items-center border border-gray-700 rounded-lg">
                                    <button onclick="window.updateCartItemQuantity(${item.id}, -1)" class="p-1 text-primary-red hover:bg-gray-700 transition duration-150 rounded-l-lg">
                                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 12H4"></path></svg>
                                    </button>
                                    <span class="px-2 text-sm font-medium text-white">${item.quantity}</span>
                                    <button onclick="window.updateCartItemQuantity(${item.id}, 1)" class="p-1 text-highlight-yellow hover:bg-gray-700 transition duration-150 rounded-r-lg">
                                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                                    </button>
                                </div>
                                <span class="text-base font-bold text-highlight-yellow w-16 text-right">
                                    R$ ${(itemPrice * item.quantity).toFixed(2).replace('.', ',')}
                                </span>
                            </div>
                        </div>
                    `;
                    container.innerHTML += itemHTML;
                });

                // Botão de Limpar Sacola
                container.innerHTML += `
                    <button onclick="window.clearCart()" class="w-full text-center text-sm text-gray-400 mt-4 hover:text-primary-red transition duration-150">
                        Remover Todos os Itens
                    </button>
                `;
            }

            // --- CÁLCULO DO FRETE ---
            let shippingFee = SHIPPING_FEE;
            let finalTotal = itemsTotal;
            let shippingMessage = `R$ ${SHIPPING_FEE.toFixed(2).replace('.', ',')}`;
            let shippingNote = '';

            if (itemsTotal >= FREE_SHIPPING_THRESHOLD) {
                shippingFee = 0;
                shippingMessage = '<span class="text-whatsapp-green font-extrabold">GRÁTIS!</span>';
                shippingNote = `<p class="text-sm text-whatsapp-green font-bold mt-2">🥳 Seu pedido tem frete GRÁTIS!</p>`;
            } else if (cart.length > 0) {
                 const remaining = FREE_SHIPPING_THRESHOLD - itemsTotal;
                 shippingNote = `<p class="text-xs text-gray-400 mt-2">Faltam R$ ${remaining.toFixed(2).replace('.', ',')} para ter FRETE GRÁTIS.</p>`;
            }
            
            finalTotal = itemsTotal + shippingFee;
            
            // Atualiza o HTML com a quebra de valores
            totalSummary.innerHTML = `
                <div class="bg-dark-bg p-4 rounded-xl shadow-lg mt-6 border border-gray-800 space-y-2">
                    <div class="flex justify-between items-center text-base">
                        <span class="text-gray-400">Subtotal dos Itens:</span>
                        <span class="text-white font-semibold">R$ ${itemsTotal.toFixed(2).replace('.', ',')}</span>
                    </div>
                    <div class="flex justify-between items-center text-base">
                        <span class="text-gray-400">Taxa de Entrega (Frete):</span>
                        <span class="font-bold">${shippingMessage}</span>
                    </div>
                    <div class="border-t border-gray-700 pt-2">
                        <div class="flex justify-between items-center text-xl font-extrabold">
                            <span class="text-white">TOTAL A PAGAR:</span>
                            <span class="text-primary-red">R$ ${finalTotal.toFixed(2).replace('.', ',')}</span>
                        </div>
                        ${shippingNote}
                    </div>
                </div>
                
                <!-- Botão de Finalizar Pedido -->
                <div class="mt-8">
                    <button onclick="window.finalizeOrder()" ${cart.length === 0 ? 'disabled' : ''} class="w-full bg-whatsapp-green text-white py-3 rounded-xl font-extrabold text-lg hover:bg-green-600 transition shadow-xl flex items-center justify-center disabled:opacity-50">
                        <svg class="w-6 h-6 mr-2" fill="currentColor" viewBox="0 0 24 24"><path d="M12 2C6.477 2 2 6.477 2 12c0 3.189 1.408 6.046 3.619 7.97l-.604 2.054 2.112-.556A9.97 9.97 0 0012 22c5.523 0 10-4.477 10-10S17.523 2 12 2zm3.87 13.928c-.185.308-.755.513-1.096.533-.31.02-.676.042-1.042-.115s-.967-.384-1.371-1.048c-.404-.664-.789-1.571-1.113-2.482-.324-.911-.007-1.42.227-1.654.21-.21.492-.513.74-.716.248-.202.39-.338.52-.693.13-.355.065-.664-.035-.92s-.36-.613-.746-.921c-.387-.308-.795-.37-.872-.39s-.144-.025-.218-.025c-.074 0-.2-.01-.3-.01s-.256.04-.393.18c-.137.14-.53.518-.53 1.258s.546 1.458.625 1.56s.904 1.41 2.185 2.53c.63.55 1.187.82 1.586.94s.642.09 1.05.055c.44-.04 1.066-.43 1.218-.845.152-.415.152-.77.106-.85s-.176-.14-.367-.23z"/></svg>
                        Finalizar Pedido (WhatsApp)
                    </button>
                </div>
            `;
        }
        
        /** Exibe um modal genérico */
        function showModal(modalId, title = '', message = '') {
            const modal = document.getElementById(modalId);
            
            if (modalId === 'info-modal') {
                document.getElementById('info-modal-title').textContent = title;
                document.getElementById('info-modal-message').textContent = message;
            }

            modal.classList.remove('hidden');
        }

        /** Oculta um modal genérico */
        function hideModal(modalId) {
            const modal = document.getElementById(modalId);
            modal.classList.add('hidden');
        }

        /** Inicializa a aplicação */
        function initApp() {
            loadStateFromStorage(); // Carrega o estado salvo (sacola e settings)
            updateCartBadge();
            setView(currentView);
        }

        // Expor funções para uso no HTML
        window.addToCart = addToCart;
        window.updateCartItemQuantity = updateCartItemQuantity;
        window.clearCart = clearCart;
        window.setView = setView;
        window.hideModal = hideModal;
        window.showModal = showModal;
        window.openAdminPanel = openAdminPanel;
        window.handleLogin = handleLogin;
        window.finalizeOrder = finalizeOrder;
        window.openEditForm = openEditForm;
        window.saveMenuItem = saveMenuItem;
        window.deleteMenuItem = deleteMenuItem;
        window.cancelEdit = cancelEdit;
        window.setAdminEditorView = setAdminEditorView;
        window.saveSettings = saveSettings;
        window.talkToOwner = talkToOwner;
        
        // Inicia a aplicação após o carregamento completo do DOM
        window.addEventListener('load', initApp);
    </script>

    <!-- ####################################################################### -->
    <!-- MODAIS DO APLICATIVO -->
    <!-- ####################################################################### -->

    <!-- Modal de Informação Genérica (Substitui alerts) -->
    <div id="info-modal" class="fixed inset-0 z-50 bg-black bg-opacity-75 hidden flex items-center justify-center p-4">
        <div class="bg-dark-bg p-6 rounded-xl shadow-2xl max-w-sm w-full border border-primary-red">
            <h3 id="info-modal-title" class="text-xl font-bold text-highlight-yellow mb-3">Info</h3>
            <p id="info-modal-message" class="text-sm text-white mb-6">Mensagem genérica.</p>
            
            <!-- Botões -->
            <div class="space-y-3">
                <!-- BOTÃO WHATSAPP -->
                <button onclick="window.talkToOwner(); window.hideModal('info-modal')" 
                        class="w-full bg-whatsapp-green text-white py-2 rounded-lg font-bold hover:bg-green-600 transition flex items-center justify-center shadow-md">
                    <svg class="w-5 h-5 mr-2" fill="currentColor" viewBox="0 0 24 24"><path d="M12 2C6.477 2 2 6.477 2 12c0 3.189 1.408 6.046 3.619 7.97l-.604 2.054 2.112-.556A9.97 9.97 0 0012 22c5.523 0 10-4.477 10-10S17.523 2 12 2zm3.87 13.928c-.185.308-.755.513-1.096.533-.31.02-.676.042-1.042-.115s-.967-.384-1.371-1.048c-.404-.664-.789-1.571-1.113-2.482-.324-.911-.007-1.42.227-1.654.21-.21.492-.513.74-.716.248-.202.39-.338.52-.693.13-.355.065-.664-.035-.92s-.36-.613-.746-.921c-.387-.308-.795-.37-.872-.39s-.144-.025-.218-.025c-.074 0-.2-.01-.3-.01s-.256.04-.393.18c-.137.14-.53.518-.53 1.258s.546 1.458.625 1.56s.904 1.41 2.185 2.53c.63.55 1.187.82 1.586.94s.642.09 1.05.055c.44-.04 1.066-.43 1.218-.845.152-.415.152-.77.106-.85s-.176-.14-.367-.23z"/></svg>
                    Falar com o Dono (WhatsApp)
                </button>
                <button onclick="window.hideModal('info-modal')" class="w-full bg-primary-red text-white py-2 rounded-lg font-bold hover:bg-red-600 transition">
                    Fechar
                </button>
            </div>
        </div>
    </div>

    <!-- Modal de Login do Administrador -->
    <div id="admin-modal" class="fixed inset-0 z-50 bg-black bg-opacity-75 hidden flex items-center justify-center p-4">
        <div class="bg-dark-bg p-6 rounded-xl shadow-2xl max-w-sm w-full border border-primary-red">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-xl font-bold text-primary-red">🔒 Acesso Administrativo</h3>
                <button onclick="window.hideModal('admin-modal')" class="text-gray-400 hover:text-white transition">
                    <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                </button>
            </div>
            <p class="text-sm text-gray-400 mb-6">Faça login para gerenciar o cardápio e os valores. (Usuário: JB delicias)</p>

            <form onsubmit="window.handleLogin(event)">
                <div class="mb-4">
                    <label for="admin-user" class="block text-sm font-medium text-white mb-1">Usuário</label>
                    <input type="text" id="admin-user" required class="w-full p-2 rounded-lg bg-gray-800 text-white border border-gray-700 focus:ring-primary-red focus:border-primary-red" placeholder="JB delicias">
                </div>
                <div class="mb-6">
                    <label for="admin-pass" class="block text-sm font-medium text-white mb-1">Senha</label>
                    <input type="password" id="admin-pass" required class="w-full p-2 rounded-lg bg-gray-800 text-white border border-gray-700 focus:ring-primary-red focus:border-primary-red" placeholder="******">
                </div>
                <p id="login-error" class="text-sm text-highlight-yellow mb-4"></p>
                <button type="submit" class="w-full bg-primary-red text-white py-2 rounded-lg font-bold hover:bg-red-600 transition">
                    Entrar
                </button>
            </form>
        </div>
    </div>

    <!-- Modal de Editor de Cardápio/Configurações (Admin Logado) -->
    <div id="menu-editor-modal" class="fixed inset-0 z-50 bg-black bg-opacity-75 hidden flex items-center justify-center p-4">
        <div class="bg-dark-bg p-6 rounded-xl shadow-2xl max-w-lg w-full h-[90vh] overflow-y-auto border border-highlight-yellow">
            
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-2xl font-bold text-highlight-yellow">🛠️ Painel de Admin (Local)</h3>
                <button onclick="window.hideModal('menu-editor-modal')" class="text-gray-400 hover:text-white transition">
                    <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                </button>
            </div>

            <!-- Navegação Interna do Admin -->
            <div class="flex bg-gray-800 rounded-lg p-1 mb-6">
                <button id="admin-menu-tab" onclick="window.setAdminEditorView('menu')" class="admin-tab-button flex-1 py-2 px-4 rounded-lg text-sm font-semibold transition bg-primary-red text-white">
                    Cardápio
                </button>
                <button id="admin-settings-tab" onclick="window.setAdminEditorView('settings')" class="admin-tab-button flex-1 py-2 px-4 rounded-lg text-sm font-semibold transition text-gray-400 hover:bg-gray-700">
                    Configurações
                </button>
            </div>


            <!-- Seção 1: Lista de Itens do Cardápio -->
            <div id="item-list-section">
                <h4 class="text-xl font-bold text-white mb-4">Gerenciar Cardápio</h4>
                <div id="admin-item-list" class="space-y-4">
                    <p class="text-center text-gray-500">Carregando itens para edição...</p>
                </div>
                <p class="text-xs text-center text-primary-red mt-4">Aviso: Mudanças no cardápio não são permanentes, serão perdidas ao recarregar a página (exceto a sacola).</p>
            </div>

            <!-- Seção 2: Edição de Item (Formulário - Oculta por padrão) -->
            <div id="edit-item-section" class="hidden">
                <h4 id="edit-form-title" class="text-xl font-bold text-primary-red mb-4">Adicionar Novo Item</h4>
                
                <form id="menu-edit-form" onsubmit="window.saveMenuItem(event)" class="space-y-4">
                    <div>
                        <label for="edit-name" class="block text-sm font-medium text-white mb-1">Nome do Lanche</label>
                        <input type="text" id="edit-name" required class="w-full p-2 rounded-lg bg-gray-800 text-white border border-gray-700 focus:ring-primary-red focus:border-primary-red">
                    </div>
                    <div>
                        <label for="edit-description" class="block text-sm font-medium text-white mb-1">Descrição</label>
                        <textarea id="edit-description" required class="w-full p-2 rounded-lg bg-gray-800 text-white border border-gray-700 focus:ring-primary-red focus:border-primary-red"></textarea>
                    </div>
                    <div>
                        <label for="edit-price" class="block text-sm font-medium text-white mb-1">Preço (R$)</label>
                        <input type="number" step="0.01" id="edit-price" required class="w-full p-2 rounded-lg bg-gray-800 text-white border border-gray-700 focus:ring-primary-red focus:border-primary-red" placeholder="Ex: 29.90">
                    </div>
                    <div>
                        <label for="edit-img" class="block text-sm font-medium text-white mb-1">URL da Imagem (Opcional)</label>
                        <input type="url" id="edit-img" class="w-full p-2 rounded-lg bg-gray-800 text-white border border-gray-700 focus:ring-primary-red focus:border-primary-red" placeholder="https://placehold.co/100x100/3B0000/FFFFFF?text=Xis">
                    </div>
                    
                    <div class="flex justify-between pt-4">
                        <button type="button" onclick="window.cancelEdit()" class="px-4 py-2 bg-gray-700 text-white rounded-lg hover:bg-gray-600 transition">
                            Cancelar
                        </button>
                        <button type="submit" class="px-4 py-2 bg-primary-red text-white rounded-lg font-bold hover:bg-red-600 transition shadow-lg">
                            Salvar Item
                        </button>
                    </div>
                </form>
            </div>
            
            <!-- Seção 3: Editor de Configurações Gerais -->
            <div id="settings-editor-section" class="hidden">
                <h4 class="text-xl font-bold text-white mb-4">Configurações Gerais (Link de Contato e QR Code)</h4>
                
                <form id="settings-edit-form" onsubmit="window.saveSettings(event)" class="space-y-4">
                    
                    <div class="bg-gray-800 p-4 rounded-lg">
                        <h5 class="font-semibold text-highlight-yellow mb-2">Link de Contato (WhatsApp)</h5>
                        <label for="edit-whatsapp-number" class="block text-sm font-medium text-white mb-1">Número de WhatsApp Completo</label>
                        <input type="text" id="edit-whatsapp-number" required class="w-full p-2 rounded-lg bg-dark-bg text-white border border-gray-700 focus:ring-primary-red focus:border-primary-red" placeholder="555181114714">
                        <p class="text-xs text-gray-500 mt-1">Este número será usado para gerar o link do seu pedido. Deve incluir o código do país (55) e o DDD (51), sem espaços ou hífens.</p>
                    </div>
                    
                    <div class="bg-gray-800 p-4 rounded-lg">
                        <h5 class="font-semibold text-highlight-yellow mb-2">Imagem de QR Code (Convite)</h5>
                        <label for="edit-qr-code-img" class="block text-sm font-medium text-white mb-1">URL de Imagem OU Código Base64</label>
                        <input type="text" id="edit-qr-code-img" class="w-full p-2 rounded-lg bg-dark-bg text-white border border-gray-700 focus:ring-primary-red focus:border-primary-red" placeholder="Cole o link (URL) ou a string Base64 aqui.">
                        <p class="text-xs text-gray-500 mt-1">Aqui você pode colocar uma URL (ex: `https://.../qrcode.png`) ou a string Base64 de uma imagem salva no seu celular (ex: `data:image/png;base64,...`).</p>
                    </div>

                    <div class="bg-gray-800 p-4 rounded-lg">
                        <h5 class="font-semibold text-highlight-yellow mb-2">Programa Convite</h5>
                        <label for="edit-invite-text" class="block text-sm font-medium text-white mb-1">Texto da Recompensa</label>
                        <textarea id="edit-invite-text" required rows="3" class="w-full p-2 rounded-lg bg-dark-bg text-white border border-gray-700 focus:ring-primary-red focus:border-primary-red" placeholder="Ex: a cada 3 amigos que comprarem você ganha..."></textarea>
                    </div>
                    
                    <div class="flex justify-end pt-4">
                        <button type="submit" class="px-4 py-2 bg-primary-red text-white rounded-lg font-bold hover:bg-red-600 transition shadow-lg">
                            Salvar Configurações
                        </button>
                    </div>
                </form>
            </div>
            
        </div>
    </div>


    <!-- ####################################################################### -->
    <!-- LAYOUT PRINCIPAL -->
    <!-- ####################################################################### -->

    <!-- Header Fixo (Topo do App) -->
    <header class="fixed top-0 left-0 right-0 z-40 bg-black shadow-lg p-4">
        <div class="max-w-md mx-auto">
            <!-- Título Principal e Configurações -->
            <div class="flex justify-between items-center mb-4">
                <div class="text-left">
                    <p class="text-xs text-white uppercase font-light">Lanchonete e Delivery</p>
                    <h1 class="text-2xl font-black text-primary-red">LANCHES DA B</h1>
                    <p class="text-base font-semibold text-highlight-yellow">JB delicias</p>
                </div>
                <!-- Botão de Configurações (Abre Admin Login/Painel) -->
                <button onclick="window.openAdminPanel()"
                        class="text-gray-400 hover:text-highlight-yellow transition duration-150 p-2 rounded-full hover:bg-gray-800">
                    <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
                </button>
            </div>

            <!-- Barra de Navegação (Tabs) -->
            <nav class="flex justify-around text-sm font-semibold border-b border-gray-800">
                <button id="menu-tab" onclick="window.setView('menu')" class="tab-button flex flex-col items-center py-2 px-3 text-gray-400 transition duration-150">
                    <svg class="w-6 h-6 mb-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 4v12l-4-2-4 2V4M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path></svg>
                    Cardápio
                </button>
                <button id="invite-tab" onclick="window.setView('invite')" class="tab-button flex flex-col items-center py-2 px-3 text-gray-400 transition duration-150">
                    <svg class="w-6 h-6 mb-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13.828 10.172a4 4 0 00-5.656 0l-4 4a4 4 0 105.656 5.656l1.102-1.101m-.758-4.808l-1.554 1.555m0 0l1.554 1.555m7.371-7.37l-1.553-1.554m0 0l1.553 1.554m-4.04-2.828a4 4 0 015.656 0l4 4a4 4 0 01-5.656 5.656l-1.1-1.1"></path></svg>
                    Convite
                </button>
                <button id="cart-tab" onclick="window.setView('cart')" class="tab-button relative flex flex-col items-center py-2 px-3 text-gray-400 transition duration-150">
                    <svg class="w-6 h-6 mb-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z"></path></svg>
                    Carrinho
                    <span id="cart-badge" class="absolute top-0 right-0 inline-flex items-center justify-center px-2 py-1 text-xs font-bold leading-none text-white transform translate-x-1/2 -translate-y-1/2 bg-primary-red rounded-full hidden">0</span>
                </button>
            </nav>
        </div>
    </header>

    <!-- Conteúdo Principal (Centralizado em Mobile) -->
    <main id="main-content" class="max-w-md mx-auto p-4 bg-black min-h-screen">
        
        <h2 id="view-title" class="text-2xl font-bold text-white mb-6 pt-2">Cardápio</h2>
        
        <!-- Seção: Cardápio -->
        <section id="menu-content" class="space-y-4">
            <div id="menu-list" class="space-y-3">
                <!-- Itens do Cardápio serão renderizados aqui pelo JS -->
                <p class="text-center text-gray-500">Carregando cardápio...</p>
            </div>
        </section>

        <!-- Seção: Convite -->
        <section id="invite-content" class="space-y-4 hidden">
            <!-- Conteúdo de Convite será renderizado aqui pelo JS -->
        </section>

        <!-- Seção: Carrinho (Sacola) -->
        <section id="cart-content" class="space-y-4 hidden">
            <div id="cart-list" class="space-y-3">
                <!-- Itens da Sacola serão renderizados aqui pelo JS -->
            </div>
            
            <!-- Resumo do Carrinho -->
            <div id="cart-summary">
                 <!-- Resumo do frete e total será renderizado aqui pelo JS -->
            </div>
        </section>

    </main>

</body>
</html>

