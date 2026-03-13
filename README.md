# herb.github.io
from flask import Flask, render_template_string, jsonify, send_from_directory
import os
import json
from pathlib import Path

app = Flask(__name__, template_folder='templates', static_folder='static')
app.config['TEMPLATES_AUTO_RELOAD'] = True

# Create directories if they don't exist
os.makedirs('static/css', exist_ok=True)
os.makedirs('static/js', exist_ok=True)
os.makedirs('static/images', exist_ok=True)

# Herbs data embedded directly
HERBS_DATA = [
    {
        "id": 0,
        "Name": "Turmeric",
        "ScientificName": "Curcuma longa",
        "Image": "/static/images/turmeric.png",
        "Description": "Turmeric is a bright yellow spice widely used in Indian cuisine and traditional medicine. It's renowned for its powerful anti-inflammatory and antioxidant properties.",
        "Uses": [
            "Reduces inflammation and joint pain",
            "Supports digestion and liver health",
            "Boosts immunity and fights infections",
            "Promotes healthy skin",
            "May help prevent cancer"
        ],
        "Precautions": [
            "May thin blood - consult doctor if on blood thinners",
            "High doses may cause stomach upset",
            "Avoid before surgery"
        ]
    },
    {
        "id": 1,
        "Name": "Ginger",
        "ScientificName": "Zingiber officinale",
        "Image": "/static/images/ginger.png",
        "Description": "Ginger is a flowering plant with a spicy, aromatic rhizome used fresh, dried, or powdered. It's one of the healthiest spices on Earth.",
        "Uses": [
            "Relieves nausea and motion sickness",
            "Reduces muscle pain and soreness",
            "Anti-inflammatory effects",
            "Supports digestion",
            "Lowers blood sugar levels"
        ],
        "precautions": [
            "May interact with blood thinners",
            "High doses may cause heartburn",
            "Avoid if you have gallstones"
        ]
    },
    {
        "id": 2,
        "name": "Garlic",
        "scientificName": "Allium sativum",
        "image": "/static/images/garlic.png",
        "description": "Garlic is a species in the onion genus with a strong odor and taste. It's been used medicinally for thousands of years.",
        "uses": [
            "Boosts immune system",
            "Lowers blood pressure",
            "Reduces cholesterol levels",
            "Antibacterial and antiviral",
            "Supports heart health"
        ],
        "precautions": [
            "May cause bad breath and body odor",
            "Can thin blood",
            "May cause stomach upset in some people"
        ]
    },
    {
        "id": 3,
        "name": "Peppermint",
        "scientificName": "Mentha × piperita",
        "image": "/static/images/peppermint.png",
        "description": "Peppermint is a hybrid mint with high menthol content, giving it a cooling sensation. It's widely used for digestive issues.",
        "uses": [
            "Relieves IBS symptoms",
            "Eases headaches and migraines",
            "Clears nasal congestion",
            "Soothes stomach cramps",
            "Freshens breath"
        ],
        "precautions": [
            "May worsen acid reflux",
            "Avoid if you have GERD",
            "Can cause heartburn in some"
        ]
    },
    {
        "id": 4,
        "name": "Aloe Vera",
        "scientificName": "Aloe barbadensis miller",
        "image": "/static/images/aloe-vera.png",
        "description": "Aloe vera is a succulent plant species containing a thick, clear gel used in healing and cosmetics.",
        "uses": [
            "Heals burns and wounds",
            "Moisturizes skin",
            "Reduces inflammation",
            "Supports digestion",
            "Boosts oral health"
        ],
        "precautions": [
            "Latex can cause diarrhea",
            "May lower blood sugar",
            "Avoid if pregnant or breastfeeding"
        ]
    },
    {
        "id": 5,
        "name": "Echinacea",
        "scientificName": "Echinacea purpurea",
        "image": "/static/images/echinacea.png",
        "description": "Echinacea is a group of herbaceous flowering plants known for immune-boosting properties.",
        "uses": [
            "Prevents and treats colds",
            "Boosts immune system",
            "Reduces inflammation",
            "Supports respiratory health",
            "May help with anxiety"
        ],
        "precautions": [
            "May cause allergic reactions",
            "Avoid if you have autoimmune disease",
            "Not for long-term use"
        ]
    },
    {
        "id": 6,
        "name": "Chamomile",
        "scientificName": "Matricaria chamomilla",
        "image": "/static/images/chamomile.png",
        "description": "Chamomile is an herb commonly used in tea for its calming effects and medicinal properties.",
        "uses": [
            "Promotes relaxation and sleep",
            "Reduces anxiety",
            "Soothes digestive issues",
            "Anti-inflammatory",
            "Eases menstrual cramps"
        ],
        "precautions": [
            "May cause allergic reactions",
            "Can interact with sedatives",
            "Avoid if allergic to ragweed"
        ]
    }
]

# HTML Template (embedded)
INDEX_HTML = '''
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Medicinal Herbs Explorer - HerbHub</title>
    <link rel="stylesheet" href="/static/css/style.css">
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">
</head>
<body>
    <header>
        <nav class="navbar">
            <div class="nav-container">
                <h1 class="logo">🌿 HerbHub</h1>
                <ul class="nav-menu">
                    <li><a href="#home" class="nav-link">Home</a></li>
                    <li><a href="#about" class="nav-link">About</a></li>
                    <li><a href="#herbs" class="nav-link">Herbs</a></li>
                </ul>
            </div>
        </nav>
    </header>

    <main>
        <section id="home" class="hero">
            <div class="hero-content">
                <h1>Discover the Power of Medicinal Herbs</h1>
                <p>Explore seven powerful herbs and their incredible health benefits</p>
                <a href="#herbs" class="cta-button">Explore Herbs</a>
            </div>
        </section>

        <section id="about" class="about">
            <div class="container">
                <h2>About Medicinal Herbs</h2>
                <p>Medicinal herbs have been used for centuries across cultures to promote health and wellness. These natural remedies offer a holistic approach to healing, supporting various aspects of physical and mental well-being. This interactive guide showcases seven remarkable herbs and their unique properties.</p>
            </div>
        </section>

        <section id="herbs" class="herbs">
            <div class="container">
                <h2>Seven Powerful Herbs</h2>
                <div class="herbs-grid" id="herbsGrid">
                    {% for herb in herbs %}
                    <div class="herb-card" onclick="showHerbModal({{ herb.id }})">
                        <div class="herb-image-placeholder" style="background: linear-gradient(135deg, #{{ '%06x' % (0x1000000 + int(herb.id * 12345) % 0xFFFFFF) }}, #{{ '%06x' % (0x1000000 + int(herb.id * 54321) % 0xFFFFFF) }});">
                            <span class="herb-icon">{{ herb.name[0] }}</span>
                        </div>
                        <h3>{{ herb.name }}</h3>
                        <p><em>{{ herb.scientificName }}</em></p>
                    </div>
                    {% endfor %}
                </div>
            </div>
        </section>

        <!-- Herb Modal -->
        <div id="herbModal" class="modal">
            <div class="modal-content">
                <span class="close" onclick="closeModal()">&times;</span>
                <div id="modalBody">
                    <!-- Herb details loaded via JavaScript -->
                </div>
            </div>
        </div>
    </main>

    <footer>
        <div class="container">
            <p>&copy; 2024 HerbHub. College Assignment Project. Powered by Python Flask. <br>Scan QR to access: http://localhost:5000</p>
        </div>
    </footer>

    <script src="/static/js/script.js"></script>
</body>
</html>
'''

# CSS (embedded and auto-saved)
CSS_CONTENT = '''
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Poppins', sans-serif;
    line-height: 1.6;
    color: #333;
    overflow-x: hidden;
}

.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
}

.navbar {
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(10px);
    position: fixed;
    top: 0;
    width: 100%;
    z-index: 1000;
    box-shadow: 0 2px 20px rgba(0,0,0,0.1);
}

.nav-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 1rem 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.logo {
    font-size: 1.8rem;
    font-weight: 700;
    color: #2d5a2d;
}

.nav-menu {
    display: flex;
    list-style: none;
    gap: 2rem;
}

.nav-link {
    text-decoration: none;
    color: #333;
    font-weight: 500;
    transition: color 0.3s ease;
}

.nav-link:hover {
    color: #2d5a2d;
}

.hero {
    height: 100vh;
    background: linear-gradient(135deg, #a8e6cf 0%, #dcedc1 100%);
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding-top: 80px;
}

.hero-content h1 {
    font-size: 3.5rem;
    margin-bottom: 1rem;
    color: #2d5a2d;
}

.hero-content p {
    font-size: 1.3rem;
    margin-bottom: 2rem;
    color: #555;
}

.cta-button {
    display: inline-block;
    background: #2d5a2d;
    color: white;
    padding: 15px 40px;
    text-decoration: none;
    border-radius: 50px;
    font-weight: 600;
    font-size: 1.1rem;
    transition: all 0.3s ease;
    box-shadow: 0 10px 30px rgba(45, 90, 45, 0.3);
}

.cta-button:hover {
    transform: translateY(-3px);
    box-shadow: 0 15px 40px rgba(45, 90, 45, 0.4);
}

.about {
    padding: 100px 0;
    background: #f8f9fa;
}

.about h2 {
    text-align: center;
    font-size: 2.5rem;
    margin-bottom: 2rem;
    color: #2d5a2d;
}

.about p {
    text-align: center;
    font-size: 1.2rem;
    max-width: 800px;
    margin: 0 auto;
    color: #666;
}

.herbs {
    padding: 100px 0;
}

.herbs h2 {
    text-align: center;
    font-size: 2.5rem;
    margin-bottom: 4rem;
    color: #2d5a2d;
}

.herbs-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
    gap: 2rem;
}

.herb-card {
    background: white;
    border-radius: 20px;
    padding: 2rem;
    text-align: center;
    box-shadow: 0 20px 60px rgba(0,0,0,0.1);
    transition: all 0.3s ease;
    cursor: pointer;
    height: 100%;
    border: 1px solid #e0e0e0;
}

.herb-card:hover {
    transform: translateY(-10px);
    box-shadow: 0 30px 80px rgba(0,0,0,0.15);
}

.herb-image-placeholder {
    width: 120px;
    height: 120px;
    margin: 0 auto 1.5rem;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

.herb-icon {
    font-size: 3rem;
    font-weight: bold;
    color: white;
    text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

.herb-card h3 {
    font-size: 1.8rem;
    margin-bottom: 1rem;
    color: #2d5a2d;
}

.herb-card p {
    color: #666;
    line-height: 1.6;
}

.modal {
    display: none;
    position: fixed;
    z-index: 2000;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0,0,0,0.8);
    backdrop-filter: blur(5px);
}

.modal-content {
    background-color: white;
    margin: 5% auto;
    padding: 0;
    border-radius: 20px;
    width: 90%;
    max-width: 800px;
    max-height: 80vh;
    overflow-y: auto;
    position: relative;
    animation: slideIn 0.3s ease;
}

@keyframes slideIn {
    from { transform: translateY(-50px); opacity: 0; }
    to { transform: translateY(0); opacity: 1; }
}

.close {
    position: absolute;
    right: 20px;
    top: 20px;
    color: #aaa;
    font-size: 28px;
    font-weight: bold;
    cursor: pointer;
    z-index: 10;
}

.close:hover {
    color: #2d5a2d;
}

.modal-header {
    padding: 2rem;
    text-align: center;
    background: linear-gradient(135deg, #2d5a2d, #4a7c4a);
    color: white;
    border-radius: 20px 20px 0 0;
}

.modal-header .herb-image-placeholder {
    width: 100px;
    height: 100px;
    margin-bottom: 1rem;
}

.modal-header h2 {
    margin-bottom: 0.5rem;
}

.modal-body {
    padding: 2rem;
}

.modal-body p {
    margin-bottom: 1.5rem;
    line-height: 1.8;
}

.uses-list, .precautions-list {
    margin: 1.5rem 0;
}

.uses-list h4, .precautions-list h4 {
    color: #2d5a2d;
    margin-bottom: 1rem;
}

.uses-list ul, .precautions-list ul {
    list-style: none;
    padding-left: 0;
}

.uses-list li, .precautions-list li {
    padding: 0.8rem 0;
    padding-left: 2rem;
    position: relative;
}

.uses-list li::before {
    content: "✔";
    position: absolute;
    left: 0;
    color: #28a745;
    font-weight: bold;
}

.precautions-list li::before {
    content: "⚠";
    position: absolute;
    left: 0;
    color: #dc3545;
}

footer {
    background: #2d5a2d;
    color: white;
    text-align: center;
    padding: 2rem 0;
}

@media (max-width: 768px) {
    .nav-menu {
        gap: 1rem;
    }
    
    .hero-content h1 {
        font-size: 2.5rem;
    }
    
    .herbs-grid {
        grid-template-columns: 1fr;
    }
    
    .modal-content {
        width: 95%;
        margin: 10% auto;
    }
}
'''

# JavaScript (embedded and auto-saved)
JS_CONTENT = '''
// Global herbs data
let herbsData = [];

// Load herbs data on page load
document.addEventListener("DOMContentLoaded", function() {
    fetch("/api/herbs")
        .then(response => response.json())
        .then(data => {
            herbsData = data;
        })
        .catch(error => console.error("Error loading herbs:", error));
});

// Show herb modal
function showHerbModal(herbId) {
    const herb = herbsData.find(h => h.id === herbId);
    if (!herb) return;

    const modalBody = document.getElementById("modalBody");
    modalBody.innerHTML = `
        <div class="modal-header">
            <div class="herb-image-placeholder" style="background: linear-gradient(135deg, #${((0x1000000 + (herb.id * 12345) % 0xFFFFFF).toString(16).slice(1))}, #${((0x1000000 + (herb.id * 54321) % 0xFFFFFF).toString(16).slice(1))});">
                <span class="herb-icon">${herb.name[0]}</span>
            </div>
            <h2>${herb.name}</h2>
            <p><strong>Scientific Name:</strong> ${herb.scientificName}</p>
        </div>
        <div class="modal-body">
            <
