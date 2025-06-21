<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Oil Order: An Interactive Investigation</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&family=Playfair+Display:wght@700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Noir & Gold -->
    <!-- Application Structure Plan: A non-linear "investigation board" SPA. The structure includes: 1. A captivating intro section. 2. A main "Investigation Board" with clickable profiles for each key figure (Diddy, Beyoncé, etc.). 3. A dedicated section explaining "The Anointing Ritual". 4. An interactive "Connection Web" chart to visualize relationships. 5. A concluding section. This dossier-style approach is chosen over a linear slideshow to empower the user to explore freely, making the discovery process more engaging and personal, which fits the conspiracy theme. Navigation is handled via a sticky header, allowing users to jump between the key pillars of the theory. -->
    <!-- Visualization & Content Choices: Report Info -> Key Figures & Connections. Goal -> Organize & Show Relationships. Viz -> HTML/CSS Profile Grid + Interactive Chart.js Scatter Plot ("Connection Web"). Interaction -> Clicking a profile reveals their detailed story in a modal-like view. Clicking a node on the chart highlights their connections. Justification -> This is more dynamic and visually compelling than static text. It allows users to see the network structure, a core concept of the report. Library/Method -> Tailwind for layout, Chart.js for the canvas-based chart. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #121212;
            color: #E0E0E0;
        }
        h1, h2, h3 {
            font-family: 'Playfair Display', serif;
        }
        .active-nav {
            color: #D4AF37;
            border-bottom: 2px solid #D4AF37;
        }
        .nav-link {
            transition: color 0.3s ease, border-bottom-color 0.3s ease;
            border-bottom: 2px solid transparent;
        }
        .nav-link:hover {
            color: #F7DC6F;
        }
        .card {
            background-color: #1E1E1E;
            border: 1px solid #333;
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }
        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.5), 0 0 15px rgba(212, 175, 55, 0.3);
        }
        .content-section {
            display: none;
        }
        .content-section.active {
            display: block;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 800px;
            height: 50vh;
            max-height: 500px;
            margin: auto;
            background-color: #1E1E1E;
            border: 1px solid #333;
            border-radius: 0.5rem;
            padding: 1rem;
        }
    </style>
</head>
<body class="antialiased">

    <header class="bg-[#181818]/80 backdrop-blur-sm sticky top-0 z-50 border-b border-[#333]">
        <nav class="container mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex items-center justify-between h-16">
                <h1 class="text-xl md:text-2xl font-bold text-white">The Oil Order</h1>
                <div class="hidden sm:flex items-center space-x-4">
                    <a href="#intro" class="nav-link px-3 py-2 text-sm font-medium">Wake Up</a>
                    <a href="#board" class="nav-link px-3 py-2 text-sm font-medium">The Board</a>
                    <a href="#ritual" class="nav-link px-3 py-2 text-sm font-medium">The Ritual</a>
                    <a href="#connections" class="nav-link px-3 py-2 text-sm font-medium">Connections</a>
                    <a href="#conclusion" class="nav-link px-3 py-2 text-sm font-medium">The Endgame</a>
                </div>
                <div class="sm:hidden">
                    <select id="mobile-nav" class="bg-[#1E1E1E] border border-[#333] text-white text-sm rounded-lg focus:ring-[#D4AF37] focus:border-[#D4AF37] block w-full p-2.5">
                        <option value="intro">Wake Up</option>
                        <option value="board">The Board</option>
                        <option value="ritual">The Ritual</option>
                        <option value="connections">Connections</option>
                        <option value="conclusion">The Endgame</option>
                    </select>
                </div>
            </div>
        </nav>
    </header>

    <main class="container mx-auto p-4 sm:p-6 lg:p-8">

        <section id="intro" class="content-section min-h-[70vh] flex items-center justify-center text-center">
            <div>
                <h2 class="text-4xl md:text-6xl font-bold tracking-tight text-white leading-tight">You thought it was just about parties?</h2>
                <p class="mt-4 text-lg md:text-xl max-w-3xl mx-auto text-gray-300">This isn’t about one man. This is about a network—a multi-decade celebrity cult hidden in plain sight. It's time to WAKE. UP. and see the truth behind the headlines. This dossier is your guide.</p>
                <button onclick="showSection('board')" class="mt-8 px-8 py-3 bg-[#D4AF37] text-black font-bold rounded-lg hover:bg-[#F7DC6F] transition-colors">Start Investigation</button>
            </div>
        </section>

        <section id="board" class="content-section">
            <div class="text-center mb-12">
                <h2 class="text-3xl md:text-4xl font-bold text-white">The Investigation Board</h2>
                <p class="mt-3 max-w-2xl mx-auto text-gray-400">Each person is a thread in a larger tapestry. Click on a profile to pull the thread and review their dossier. The connections may not be obvious at first glance.</p>
            </div>
            <div id="person-details" class="mb-12 bg-[#1E1E1E] border border-[#333] rounded-lg p-6 hidden">
                <button onclick="hideDetails()" class="float-right text-2xl font-bold hover:text-[#D4AF37]">&times;</button>
                <h3 id="details-name" class="text-2xl font-bold text-[#D4AF37] mb-2"></h3>
                <p id="details-role" class="italic text-gray-400 mb-4"></p>
                <p id="details-text" class="text-gray-300 whitespace-pre-line"></p>
            </div>
            <div id="profiles-grid" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-5 gap-6">
            </div>
        </section>

        <section id="ritual" class="content-section">
            <div class="text-center mb-12">
                <h2 class="text-3xl md:text-4xl font-bold text-white">The Anointing Ritual</h2>
                <p class="mt-3 max-w-2xl mx-auto text-gray-400">The core of the Oil Order's power isn't money or fame, but a binding ceremony disguised as a party favor. This section breaks down the supposed meaning behind the baby oil.</p>
            </div>
            <div class="max-w-4xl mx-auto grid md:grid-cols-2 gap-8 items-center">
                <div class="space-y-4">
                    <h3 class="text-2xl font-bold text-[#D4AF37]">🧴 Not Just a Meme</h3>
                    <p>In elite circles, baby oil isn't a product. It's a ritual tool used during "baptisms." Aspiring artists are "cleansed" of their innocence and inducted into the Oil Order. Diddy doesn’t throw parties; he hosts ceremonies.</p>
                    <h3 class="text-2xl font-bold text-[#D4AF37]">Babylonian Sigils</h3>
                    <p>The oil is allegedly used in specific patterns traced on the skin. These patterns correspond to ancient Babylonian sigils meant to bind loyalty, silence, and transfer energy. Once you're oiled, you’re owned.</p>
                </div>
                <div class="flex items-center justify-center p-8 bg-[#1E1E1E] rounded-lg border border-[#333]">
                    <div class="text-center">
                        <span class="text-6xl text-[#D4AF37]">🧴</span>
                        <div class="text-4xl text-white font-mono my-4">+</div>
                        <div class="text-2xl text-[#D4AF37] font-serif italic">"Baptism"</div>
                        <div class="text-4xl text-white font-mono my-4">=</div>
                        <div class="text-2xl text-red-500 font-bold">OWNED</div>
                    </div>
                </div>
            </div>
        </section>

        <section id="connections" class="content-section">
            <div class="text-center mb-8">
                <h2 class="text-3xl md:text-4xl font-bold text-white">The Connection Web</h2>
                <p class="mt-3 max-w-2xl mx-auto text-gray-400">This web visualizes the alleged network. Click on any node to highlight their direct connections and see how they fit into the larger structure. The lines represent influence, collaboration, or conflict.</p>
            </div>
            <div class="chart-container">
                <canvas id="connectionChart"></canvas>
            </div>
             <p id="connection-info" class="text-center mt-4 text-gray-400 h-6"></p>
        </section>
        
        <section id="conclusion" class="content-section">
            <div class="text-center mb-12">
                <h2 class="text-3xl md:text-4xl font-bold text-white">The Endgame: Damage Control</h2>
                <p class="mt-3 max-w-2xl mx-auto text-gray-400">Why is this all coming out now? The official story is justice. The theory suggests a different motive: the cult is collapsing, and this is a desperate attempt at damage control.</p>
            </div>
            <div class="max-w-3xl mx-auto bg-[#1E1E1E] p-8 rounded-lg border border-[#333]">
                <p class="text-lg mb-6">Diddy’s raids and court cases aren't the beginning of the end. They are the public face of an internal power struggle. As ex-members flip and evidence surfaces, the elite are panicking. The silence from top-tier figures like Beyoncé isn't a lack of opinion—it's a sign of their inability to speak.</p>
                <div class="border-t border-[#333] pt-6 space-y-4">
                    <p class="italic">“The industry ain’t what it seems.” – Michael Jackson</p>
                    <p class="italic">“You don’t know what I’ve seen.” – Kanye West</p>
                    <p class="italic">“This is a performance art piece.” – Lady Gaga</p>
                    <p class="italic font-bold text-[#D4AF37]">“Let’s take that walk…” – Diddy</p>
                </div>
                <h3 class="mt-8 text-2xl font-bold text-center">Are you going to stay asleep?</h3>
            </div>
        </section>

    </main>

    <script>
        const sections = document.querySelectorAll('.content-section');
        const navLinks = document.querySelectorAll('.nav-link');
        const mobileNav = document.getElementById('mobile-nav');
        const profilesGrid = document.getElementById('profiles-grid');
        const detailsView = document.getElementById('person-details');
        const detailsName = document.getElementById('details-name');
        const detailsRole = document.getElementById('details-role');
        const detailsText = document.getElementById('details-text');

        const peopleData = [
            {
                name: "Diddy",
                role: "The Patriarch",
                text: "You thought he was just a producer with legal issues? The theory posits him as the host of the 'Anointing' ceremonies. His parties are rituals. The baby oil is a tool of control, used to trace Babylonian sigils that bind loyalty and silence. He doesn't just make stars; he owns them.",
                color: '#D32F2F'
            },
            {
                name: "Beyoncé",
                role: "The High Priestess",
                text: "Her 2013 Super Bowl show wasn't a performance; it was a coded ritual. The triangle hand sign wasn't for the Illuminati, but the 'Trinity of Anointing': Body, Brand, and Blood. She allegedly ascended to matron of the cult after Aaliyah’s death—a death some claim was a sacrifice. She's not a puppet; she's a queen on the board.",
                color: '#D4AF37'
            },
            {
                name: "Michael Jackson",
                role: "The Whistleblower",
                text: "MJ knew. His obsession with owning his music catalog was because he learned it was being used to embed ritual frequencies. When he said 'They don’t care about us,' he meant the cult. His 'This Is It' tour was meant to expose everything. He died before he could. Diddy-connected entities later acquired rights to his work.",
                color: '#1976D2'
            },
            {
                name: "Kanye West",
                role: "The Broken Pawn",
                text: "He was initiated but something snapped when he refused the full 'baptism.' His public meltdowns are not just mental illness, but a controlled demolition. The cult allows him to speak half-truths, making him look insane so the real secrets remain buried. It's a classic gaslighting operation.",
                color: '#7B1FA2'
            },
            {
                name: "Lady Gaga",
                role: "The Image Controller",
                text: "Rising to fame right after MJ's death was no coincidence. Her 'fame monster' persona is a protocol to train new artists in emotional dissociation and ritual submission. When she said she 'absorbed' a friend's soul to become famous, the theory claims it wasn't a metaphor. It was an admission.",
                color: '#00796B'
            }
        ];

        function showSection(id) {
            sections.forEach(section => {
                if (section.id === id) {
                    section.classList.add('active');
                } else {
                    section.classList.remove('active');
                }
            });
            navLinks.forEach(link => {
                if (link.getAttribute('href') === `#${id}`) {
                    link.classList.add('active-nav');
                } else {
                    link.classList.remove('active-nav');
                }
            });
             if (mobileNav.value !== id) {
                mobileNav.value = id;
            }
            window.scrollTo({ top: 0, behavior: 'smooth' });
            hideDetails();
        }
        
        function populateProfiles() {
            peopleData.forEach(person => {
                const card = document.createElement('div');
                card.className = 'card rounded-lg p-6 text-center cursor-pointer';
                card.innerHTML = `
                    <h3 class="text-xl font-bold text-white">${person.name}</h3>
                    <p class="text-sm text-[#D4AF37] italic">${person.role}</p>
                `;
                card.addEventListener('click', () => showDetails(person));
                profilesGrid.appendChild(card);
            });
        }
        
        function showDetails(person) {
            detailsName.textContent = person.name;
            detailsRole.textContent = person.role;
            detailsText.textContent = person.text;
            detailsView.classList.remove('hidden');
            detailsView.scrollIntoView({ behavior: 'smooth', block: 'center' });
        }

        function hideDetails() {
            detailsView.classList.add('hidden');
        }

        function setupNav() {
            const handleNav = (e) => {
                e.preventDefault();
                const id = e.currentTarget.getAttribute('href').substring(1);
                showSection(id);
            };
            navLinks.forEach(link => link.addEventListener('click', handleNav));
            mobileNav.addEventListener('change', (e) => showSection(e.target.value));
        }
        
        function setupInitialView() {
            const hash = window.location.hash.substring(1);
            const validHashes = ['intro', 'board', 'ritual', 'connections', 'conclusion'];
            if (validHashes.includes(hash)) {
                showSection(hash);
            } else {
                showSection('intro');
            }
        }

        document.addEventListener('DOMContentLoaded', () => {
            populateProfiles();
            setupNav();
            setupInitialView();
            renderConnectionChart();
        });

        function renderConnectionChart() {
            const ctx = document.getElementById('connectionChart').getContext('2d');
            const infoEl = document.getElementById('connection-info');
            
            const chartData = {
                datasets: [{
                    label: 'Key Figures',
                    data: [
                        { x: 2, y: 5, r: 20, name: 'Diddy' }, // Central figure
                        { x: 4, y: 7, r: 15, name: 'Beyoncé' }, // High priestess
                        { x: 6, y: 3, r: 15, name: 'Michael Jackson' }, // Whistleblower
                        { x: 1, y: 2, r: 15, name: 'Kanye West' }, // Pawn
                        { x: 8, y: 6, r: 15, name: 'Lady Gaga' }, // Handler
                    ],
                    backgroundColor: peopleData.map(p => p.color),
                    borderColor: '#FFFFFF',
                    borderWidth: 1,
                }]
            };

            const connections = {
                'Diddy': ['Beyoncé', 'Michael Jackson', 'Kanye West'],
                'Beyoncé': ['Diddy', 'Aaliyah'],
                'Michael Jackson': ['Diddy', 'Sony'],
                'Kanye West': ['Diddy'],
                'Lady Gaga': ['Michael Jackson']
            };

            const chart = new Chart(ctx, {
                type: 'bubble',
                data: chartData,
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    onClick: (evt, elements) => {
                        if (elements.length > 0) {
                            const elementIndex = elements[0].index;
                            const person = peopleData[elementIndex];
                            infoEl.textContent = `${person.name}'s connections: ${connections[person.name] ? connections[person.name].join(', ') : 'None listed'}.`;
                            infoEl.style.color = person.color;

                            chart.data.datasets[0].borderColor = chart.data.datasets[0].backgroundColor.map((_, i) => i === elementIndex ? person.color : '#FFFFFF');
                            chart.data.datasets[0].borderWidth = chart.data.datasets[0].backgroundColor.map((_, i) => i === elementIndex ? 4 : 1);
                            
                            chart.update();

                        } else {
                            infoEl.textContent = 'Click a node to see their connections.';
                            infoEl.style.color = '#9CA3AF';
                            chart.data.datasets[0].borderColor = '#FFFFFF';
                            chart.data.datasets[0].borderWidth = 1;
                            chart.update();
                        }
                    },
                    plugins: {
                        legend: {
                            display: false
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    return context.raw.name;
                                }
                            }
                        }
                    },
                    scales: {
                        x: {
                            display: false,
                            min: 0,
                            max: 10
                        },
                        y: {
                            display: false,
                            min: 0,
                            max: 10
                        }
                    }
                }
            });
        }

    </script>
</body>
</html>

