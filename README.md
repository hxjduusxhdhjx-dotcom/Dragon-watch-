<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DRAGON WATCH | إمبراطورية الأنمي 🇩🇿</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com">
    <style>
        @import url('https://fonts.googleapis.com');
        :root { --dragon-red: #ff0000; --deep-black: #050505; }
        body { font-family: 'Cairo', sans-serif; background: var(--deep-black); color: #fff; overflow-x: hidden; }
        .glass-card { background: rgba(15, 15, 15, 0.85); backdrop-filter: blur(15px); border: 1px solid rgba(255,255,255,0.05); }
        .movie-card { transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); position: relative; overflow: hidden; border-radius: 15px; cursor: pointer; }
        .movie-card:hover { transform: scale(1.05); box-shadow: 0 0 25px rgba(255,0,0,0.4); }
        .episode-btn.active { background: var(--dragon-red) !important; border-color: #fff !important; transform: scale(1.05); }
        .server-btn.active { background: #fff !important; color: #000 !important; }
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-thumb { background: var(--dragon-red); border-radius: 10px; }
        iframe { border: none; width: 100%; height: 100%; border-radius: 15px; background: #000; }
    </style>
</head>
<body>

    <!-- Loader -->
    <div id="loader" class="fixed inset-0 bg-black z-[9999] flex flex-col justify-center items-center">
        <i class="fas fa-dragon text-7xl text-red-600 animate-bounce"></i>
        <h2 class="text-xl font-bold mt-5 italic">جارٍ تحميل القوة...</h2>
    </div>

    <!-- Navigation -->
    <nav class="sticky top-0 z-[100] glass-card p-4">
        <div class="container mx-auto flex justify-between items-center gap-4">
            <div class="flex items-center gap-3 cursor-pointer" onclick="location.reload()">
                <i class="fas fa-dragon text-3xl text-red-600"></i>
                <h1 class="text-2xl font-black italic hidden sm:block">DRAGON <span class="text-red-600">WATCH</span></h1>
            </div>
            <div class="flex-1 max-w-xl relative">
                <input type="text" id="searchInput" oninput="debounceSearch()" placeholder="ابحث عن أنمي أو فيلم..." class="w-full bg-white/5 border border-white/10 rounded-full py-2.5 px-12 focus:border-red-600 outline-none transition-all">
                <i class="fas fa-search absolute right-5 top-1/2 -translate-y-1/2 text-zinc-500"></i>
            </div>
        </div>
    </nav>

    <!-- Main Content -->
    <main class="container mx-auto px-4 mt-8 pb-20">
        <div class="flex justify-center mb-10">
            <div class="flex gap-2 bg-zinc-900 p-1 rounded-full border border-white/5">
                <button onclick="changeMode('anime')" id="btn-anime" class="mode-btn px-8 py-2 rounded-full font-bold bg-red-600">أنمي</button>
                <button onclick="changeMode('movie')" id="btn-movie" class="mode-btn px-8 py-2 rounded-full font-bold">أفلام</button>
            </div>
        </div>

        <div id="main-grid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 xl:grid-cols-6 gap-6"></div>
        
        <div class="flex justify-center mt-12">
            <button onclick="loadMore()" class="px-10 py-3 bg-zinc-800 border border-white/10 rounded-full font-bold hover:bg-red-600 transition-all">إظهار المزيد 🔥</button>
        </div>
    </main>

    <!-- Player Modal -->
    <div id="player-modal" class="fixed inset-0 z-[200] bg-black/95 hidden overflow-y-auto p-4 sm:p-8">
        <div class="max-w-7xl mx-auto">
            <button onclick="closePlayer()" class="mb-6 text-xl hover:text-red-600 transition-all"><i class="fas fa-times ml-2"></i> إغلاق</button>
            
            <div class="grid lg:grid-cols-4 gap-8">
                <div class="lg:col-span-3">
                    <div class="aspect-video bg-black rounded-2xl overflow-hidden shadow-2xl border border-white/10">
                        <iframe id="main-iframe" src="" allowfullscreen></iframe>
                    </div>
                    <div class="mt-6">
                        <h2 id="video-title" class="text-3xl font-black text-red-600 mb-4"></h2>
                        <div id="servers-list" class="flex flex-wrap gap-2 mb-6"></div>
                        <p id="media-desc" class="text-zinc-400 leading-relaxed"></p>
                    </div>
                </div>

                <div id="series-panel" class="hidden">
                    <div class="glass-card p-5 rounded-2xl h-fit max-h-[700px] flex flex-col">
                        <h3 class="font-bold mb-4 flex items-center gap-2"><i class="fas fa-list text-red-600"></i> قائمة الحلقات</h3>
                        <select id="season-select" onchange="loadEpisodes(this.value)" class="bg-zinc-800 border border-white/10 p-3 rounded-xl mb-4 outline-none w-full text-sm"></select>
                        <div id="episodes-grid" class="grid grid-cols-4 gap-2 overflow-y-auto pr-2"></div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        const API_KEY = '05a5ee826b49b43e6b2a86822cc97e40';
        let currentMode = 'anime', currentPage = 1, activeId, activeS = 1, activeE = 1, searchTimer;

        const servers = [
            { name: "سيرفر التنين", url: "https://vidsrc.pro{type}/{id}?season={s}&episode={e}" },
            { name: "سيرفر VIP", url: "https://vidsrc.me{type}?tmdb={id}&s={s}&e={e}" },
            { name: "سيرفر خارجي", url: "https://embed.su{type}/{id}/{s}/{e}" }
        ];

        window.onload = () => {
            fetchData();
            setTimeout(() => document.getElementById('loader').style.display = 'none', 1000);
        };

        async function fetchData(isNew = true) {
            if (isNew) { currentPage = 1; document.getElementById('main-grid').innerHTML = ''; }
            const query = document.getElementById('searchInput').value.trim();
            const type = currentMode === 'anime' ? 'tv' : 'movie';
            const genre = currentMode === 'anime' ? '&with_genres=16' : '';
            
            let url = query 
                ? `https://api.themoviedb.org{type}?api_key=${API_KEY}&query=${query}&language=ar`
                : `https://api.themoviedb.org{type}?api_key=${API_KEY}${genre}&sort_by=popularity.desc&page=${currentPage}&language=ar`;

            const res = await fetch(url);
            const data = await res.json();
            renderItems(data.results);
        }

        function renderItems(items) {
            const grid = document.getElementById('main-grid');
            items.forEach(item => {
                if (!item.poster_path) return;
                const div = document.createElement('div');
                div.className = "movie-card animate__animated animate__fadeIn";
                div.innerHTML = `
                    <img src="https://image.tmdb.org{item.poster_path}" class="w-full h-[300px] object-cover">
                    <div class="absolute inset-0 bg-gradient-to-t from-black flex items-end p-4">
                        <h3 class="text-sm font-bold truncate">${item.name || item.title}</h3>
                    </div>
                `;
                div.onclick = () => openMedia(item);
                grid.appendChild(div);
            });
        }

        async function openMedia(item) {
            activeId = item.id;
            const modal = document.getElementById('player-modal');
            modal.classList.remove('hidden');
            document.getElementById('video-title').innerText = item.name || item.title;
            document.getElementById('media-desc').innerText = item.overview || "لا يوجد وصف حالياً.";
            
            if (currentMode === 'anime') {
                document.getElementById('series-panel').classList.remove('hidden');
                await loadSeasons(item.id);
            } else {
                document.getElementById('series-panel').classList.add('hidden');
                updatePlayer('movie', item.id, 1, 1);
            }
        }

        async function loadSeasons(id) {
            const res = await fetch(`https://api.themoviedb.org{id}?api_key=${API_KEY}&language=ar`);
            const data = await res.json();
            const select = document.getElementById('season-select');
            select.innerHTML = '';
            
            data.seasons.filter(s => s.season_number > 0).forEach(s => {
                select.innerHTML += `<option value="${s.season_number}">الموسم ${s.season_number}</option>`;
            });
            loadEpisodes(select.value || 1);
        }

        async function loadEpisodes(sNum) {
            activeS = sNum;
            const res = await fetch(`https://api.themoviedb.org{activeId}/season/${sNum}?api_key=${API_KEY}&language=ar`);
            const data = await res.json();
            const grid = document.getElementById('episodes-grid');
            grid.innerHTML = '';

            data.episodes.forEach(ep => {
                const btn = document.createElement('button');
                btn.className = `episode-btn p-2 rounded bg-white/5 border border-white/10 text-xs hover:bg-red-600 transition-all ${activeE == ep.episode_number ? 'active' : ''}`;
                btn.innerText = ep.episode_number;
                btn.onclick = () => {
                    activeE = ep.episode_number;
                    updatePlayer('tv', activeId, activeS, activeE);
                    document.querySelectorAll('.episode-btn').forEach(b => b.classList.remove('active'));
                    btn.classList.add('active');
                };
                grid.appendChild(btn);
            });
            updatePlayer('tv', activeId, activeS, 1);
        }

        function updatePlayer(type, id, s, e) {
            renderServers(type, id, s, e);
            const url = servers[0].url.replace('{type}', type).replace('{id}', id).replace('{s}', s).replace('{e}', e);
            document.getElementById('main-iframe').src = url;
        }

        function renderServers(type, id, s, e) {
            const container = document.getElementById('servers-list');
            container.innerHTML = '';
            servers.forEach((srv, i) => {
                const btn = document.createElement('button');
                btn.className = `server-btn px-4 py-2 rounded-lg bg-zinc-800 text-xs font-bold hover:bg-white hover:text-black transition-all ${i==0?'active':''}`;
                btn.innerText = srv.name;
                btn.onclick = () => {
                    document.getElementById('main-iframe').src = srv.url.replace('{type}', type).replace('{id}', id).replace('{s}', s).replace('{e}', e);
                    document.querySelectorAll('.server-btn').forEach(b => b.classList.remove('active'));
                    btn.classList.add('active');
                };
                container.appendChild(btn);
            });
        }

        function changeMode(mode) {
            currentMode = mode;
            document.querySelectorAll('.mode-btn').forEach(b => b.classList.remove('bg-red-600'));
            document.getElementById(`btn-${mode}`).classList.add('bg-red-600');
            fetchData();
        }

        function debounceSearch() {
            clearTimeout(searchTimer);
            searchTimer = setTimeout(fetchData, 500);
        }

        function loadMore() { currentPage++; fetchData(false); }
        function closePlayer() { 
            document.getElementById('player-modal').classList.add('hidden'); 
            document.getElementById('main-iframe').src = ''; 
        }
    </script>
</body>
</html>
