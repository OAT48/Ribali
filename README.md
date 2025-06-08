<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ตารางอ่านหนังสือส่วนตัว</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Warm Neutral -->
    <!-- Application Structure Plan: The application is structured around a central dashboard for at-a-glance insights and a tabbed navigation system to switch between different book lists ('กำลังอ่าน', 'ทั้งหมด', 'อ่านจบแล้ว'), mimicking the user-friendly view-switching of Notion. A modal is used for adding and editing books to keep the main interface clean. This task-oriented design (viewing stats, managing lists, adding books) was chosen for its intuitive user flow, allowing users to quickly access the information or function they need without being overwhelmed. -->
    <!-- Visualization & Content Choices: Key stats are presented in card-like elements for quick information digestion. A donut chart (Chart.js) visualizes the 'Books by Category' data, as it's excellent for showing proportional relationships. The main book list is an interactive table (HTML/JS) for clear organization and easy updates, with progress bars for at-a-glance tracking. Checkboxes for daily goals provide a simple, direct interaction. New features include LLM-powered summary generation for books (enhancing reflection) and weekly reading goal suggestions (promoting continuous engagement). This approach transforms the static report fields into a dynamic, interactive experience, augmented by AI for deeper insights and personalized guidance. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Sarabun', sans-serif;
            background-color: #FDFBF8;
            color: #4A4A4A;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 350px;
            margin-left: auto;
            margin-right: auto;
            height: 350px;
        }
        .progress-bar-bg {
            background-color: #EAEAEA;
            border-radius: 9999px;
            overflow: hidden;
        }
        .progress-bar-fg {
            background-color: #8DBFBC;
            transition: width 0.5s ease-in-out;
        }
        .modal-bg {
            transition: opacity 0.3s ease;
        }
        .modal-content {
            transition: transform 0.3s ease;
        }
        .nav-button-active {
            background-color: #8DBFBC;
            color: white;
        }
        .nav-button-inactive {
            background-color: #FFFFFF;
            color: #4A4A4A;
        }
        .day-checkbox:checked + label {
            background-color: #A8D8D5;
            border-color: #8DBFBC;
        }
    </style>
</head>
<body class="antialiased">

    <div id="app" class="container mx-auto p-4 md:p-8 max-w-7xl">

        <header class="text-center mb-8">
            <h1 class="text-4xl font-bold text-[#3B6663]">ตารางอ่านหนังสือส่วนตัว 📚</h1>
            <p class="text-lg mt-2 text-gray-600">ติดตามและสร้างนิสัยการอ่านของคุณ</p>
        </header>
        
        <section id="dashboard" class="mb-8">
            <h2 class="text-2xl font-bold mb-4 text-[#3B6663]">ภาพรวมของคุณ</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                <div class="bg-white p-6 rounded-xl shadow-md flex items-center justify-between">
                    <div>
                        <p class="text-gray-500">กำลังอ่าน</p>
                        <p id="stat-reading" class="text-3xl font-bold">0</p>
                    </div>
                    <span class="text-4xl">⏳</span>
                </div>
                <div class="bg-white p-6 rounded-xl shadow-md flex items-center justify-between">
                    <div>
                        <p class="text-gray-500">อ่านจบแล้ว</p>
                        <p id="stat-finished" class="text-3xl font-bold">0</p>
                    </div>
                     <span class="text-4xl">✅</span>
                </div>
                <div class="bg-white p-6 rounded-xl shadow-md flex items-center justify-between">
                    <div>
                        <p class="text-gray-500">หนังสือทั้งหมด</p>
                        <p id="stat-total" class="text-3xl font-bold">0</p>
                    </div>
                     <span class="text-4xl">📖</span>
                </div>
                <div class="bg-white p-6 rounded-xl shadow-md flex items-center justify-between">
                     <div>
                        <p class="text-gray-500">คะแนนเฉลี่ย</p>
                        <p id="stat-avg-rating" class="text-3xl font-bold">0.0</p>
                    </div>
                    <span class="text-4xl">⭐</span>
                </div>
                <div class="bg-white p-6 rounded-xl shadow-md flex items-center justify-center col-span-1 md:col-span-2 lg:col-span-4">
                    <button id="generate-weekly-goals-button" onclick="generateWeeklyGoals()" class="bg-[#3B6663] hover:bg-[#2A4B49] text-white font-bold py-3 px-6 rounded-lg shadow-lg transition-transform transform hover:scale-105">
                        ✨ ไอเดียเป้าหมายการอ่านรายสัปดาห์
                    </button>
                </div>
            </div>
        </section>

        <section class="grid grid-cols-1 lg:grid-cols-3 gap-8 mb-8">
            <div class="lg:col-span-2 bg-white p-6 rounded-xl shadow-md">
                 <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4">
                    <h2 class="text-2xl font-bold text-[#3B6663] mb-2 sm:mb-0">ห้องสมุดของฉัน</h2>
                    <div id="nav-buttons" class="flex space-x-2">
                        <button data-view="กำลังอ่าน" class="nav-button px-4 py-2 rounded-lg font-semibold shadow-sm">กำลังอ่าน</button>
                        <button data-view="ทั้งหมด" class="nav-button px-4 py-2 rounded-lg font-semibold shadow-sm">ทั้งหมด</button>
                        <button data-view="อ่านจบแล้ว" class="nav-button px-4 py-2 rounded-lg font-semibold shadow-sm">อ่านจบแล้ว</button>
                    </div>
                </div>
                <div id="book-list" class="overflow-x-auto">
                </div>
                 <div class="text-center mt-6">
                    <button onclick="openModal()" class="bg-[#3B6663] hover:bg-[#2A4B49] text-white font-bold py-3 px-6 rounded-lg shadow-lg transition-transform transform hover:scale-105">
                        + เพิ่มหนังสือใหม่
                    </button>
                </div>
            </div>
            <div class="bg-white p-6 rounded-xl shadow-md">
                 <h2 class="text-2xl font-bold mb-4 text-[#3B6663] text-center">ประเภทหนังสือที่อ่านจบ</h2>
                 <div class="chart-container">
                    <canvas id="categoryChart"></canvas>
                </div>
            </div>
        </section>
    </div>

    <!-- Modal -->
    <div id="modal" class="fixed inset-0 bg-black bg-opacity-50 modal-bg flex items-center justify-center p-4 z-50 hidden">
        <div class="bg-white rounded-2xl shadow-2xl p-8 w-full max-w-2xl max-h-[90vh] overflow-y-auto modal-content transform scale-95">
            <h2 id="modal-title" class="text-2xl font-bold mb-6">เพิ่มหนังสือใหม่</h2>
            <form id="book-form" class="space-y-4">
                <input type="hidden" id="book-id">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label for="name" class="block text-sm font-medium text-gray-700">ชื่อหนังสือ</label>
                        <input type="text" id="name" required class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-[#8DBFBC] focus:border-[#8DBFBC]">
                    </div>
                    <div>
                        <label for="author" class="block text-sm font-medium text-gray-700">ผู้แต่ง</label>
                        <input type="text" id="author" required class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-[#8DBFBC] focus:border-[#8DBFBC]">
                    </div>
                </div>
                 <div>
                    <label for="category" class="block text-sm font-medium text-gray-700">ประเภท</label>
                    <input type="text" id="category" placeholder="เช่น วรรณกรรม, พัฒนาตนเอง" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-[#8DBFBC] focus:border-[#8DBFBC]">
                </div>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label for="totalPages" class="block text-sm font-medium text-gray-700">จำนวนหน้าทั้งหมด</label>
                        <input type="number" id="totalPages" min="0" required class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-[#8DBFBC] focus:border-[#8DBFBC]">
                    </div>
                    <div>
                        <label for="pagesRead" class="block text-sm font-medium text-gray-700">หน้าที่อ่านแล้ว</label>
                        <input type="number" id="pagesRead" min="0" required class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-[#8DBFBC] focus:border-[#8DBFBC]">
                    </div>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label for="status" class="block text-sm font-medium text-gray-700">สถานะ</label>
                        <select id="status" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-[#8DBFBC] focus:border-[#8DBFBC]">
                            <option value="ยังไม่อ่าน">ยังไม่อ่าน 📖</option>
                            <option value="กำลังอ่าน">กำลังอ่าน ⏳</option>
                            <option value="อ่านจบแล้ว">อ่านจบแล้ว ✅</option>
                            <option value="พักไว้ก่อน">พักไว้ก่อน ⏸️</option>
                        </select>
                    </div>
                     <div>
                        <label for="rating" class="block text-sm font-medium text-gray-700">คะแนน (1-5)</label>
                        <input type="number" id="rating" min="1" max="5" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-[#8DBFBC] focus:border-[#8DBFBC]">
                    </div>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                     <div>
                        <label for="startDate" class="block text-sm font-medium text-gray-700">วันที่เริ่มอ่าน</label>
                        <input type="date" id="startDate" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-[#8DBFBC] focus:border-[#8DBFBC]">
                    </div>
                     <div>
                        <label for="endDate" class="block text-sm font-medium text-gray-700">วันที่อ่านจบ</label>
                        <input type="date" id="endDate" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-[#8DBFBC] focus:border-[#8DBFBC]">
                    </div>
                </div>
                 <div>
                    <label for="summary" class="block text-sm font-medium text-gray-700">สรุป/ข้อคิด</label>
                    <textarea id="summary" rows="4" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-[#8DBFBC] focus:border-[#8DBFBC]"></textarea>
                    <button type="button" id="generate-summary-button" onclick="generateSummary()" class="mt-2 bg-[#A8D8D5] hover:bg-[#72a8a5] text-[#3B6663] hover:text-white font-bold py-2 px-3 rounded-lg text-sm transition-transform transform hover:scale-105">
                        ✨ สร้างสรุปอัตโนมัติ
                    </button>
                </div>

                <div class="flex justify-end space-x-4 pt-4">
                    <button type="button" onclick="closeModal()" class="bg-gray-200 hover:bg-gray-300 text-gray-800 font-bold py-2 px-4 rounded-lg">ยกเลิก</button>
                    <button type="submit" class="bg-[#8DBFBC] hover:bg-[#72a8a5] text-white font-bold py-2 px-4 rounded-lg">บันทึก</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Custom Message Box for LLM outputs -->
    <div id="custom-message-box" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-[999] hidden">
        <div class="bg-white rounded-xl shadow-2xl p-6 w-full max-w-sm">
            <h3 id="message-box-title" class="text-xl font-bold mb-4 text-[#3B6663]"></h3>
            <p id="message-box-content" class="text-gray-700 mb-6"></p>
            <div class="text-right">
                <button onclick="document.getElementById('custom-message-box').classList.add('hidden')" class="bg-[#8DBFBC] hover:bg-[#72a8a5] text-white font-bold py-2 px-4 rounded-lg">ตกลง</button>
            </div>
        </div>
    </div>

    <script>
        const sampleData = [
            { id: 1, name: 'Atomic Habits', author: 'James Clear', category: 'พัฒนาตนเอง', totalPages: 320, pagesRead: 150, status: 'กำลังอ่าน', rating: null, startDate: '2025-06-01', endDate: null, summary: 'สร้างนิสัยเล็กๆ เพื่อการเปลี่ยนแปลงที่ยิ่งใหญ่', dailyGoals: {mon: true, tue: false, wed: true, thu: false, fri: false, sat: false, sun: false} },
            { id: 2, name: 'Sapiens: A Brief History of Humankind', author: 'Yuval Noah Harari', category: 'ประวัติศาสตร์', totalPages: 464, pagesRead: 464, status: 'อ่านจบแล้ว', rating: 5, startDate: '2025-05-10', endDate: '2025-05-28', summary: 'ประวัติศาสตร์ย่อของมนุษยชาติที่น่าทึ่ง', dailyGoals: {mon: false, tue: false, wed: false, thu: false, fri: false, sat: false, sun: false} },
            { id: 3, name: 'The Psychology of Money', author: 'Morgan Housel', category: 'ธุรกิจ/การเงิน', totalPages: 256, pagesRead: 256, status: 'อ่านจบแล้ว', rating: 4, startDate: '2025-04-01', endDate: '2025-04-15', summary: 'ข้อคิดเรื่องเงินและจิตวิทยา', dailyGoals: {mon: false, tue: false, wed: false, thu: false, fri: false, sat: false, sun: false} },
            { id: 4, name: 'Dune', author: 'Frank Herbert', category: 'วรรณกรรม', totalPages: 412, pagesRead: 0, status: 'ยังไม่อ่าน', rating: null, startDate: null, endDate: null, summary: '', dailyGoals: {mon: false, tue: false, wed: false, thu: false, fri: false, sat: false, sun: false} }
        ];

        let books = []; 
        let currentView = 'กำลังอ่าน';
        let categoryChart;

        const bookListEl = document.getElementById('book-list');
        const modal = document.getElementById('modal');
        const modalTitle = document.getElementById('modal-title');
        const bookForm = document.getElementById('book-form');
        const navButtons = document.getElementById('nav-buttons');
        // Get references to modal background and content elements once after DOM is loaded
        let modalBg;
        let modalContent;

        // Gemini API configuration
        const GEMINI_API_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=";

        // Custom function to call Gemini API
        async function callGeminiAPI(prompt, isStructured = false, schema = null) {
            const apiKey = ""; // API key is empty string, Canvas will provide it at runtime.
            const apiUrl = `${GEMINI_API_URL}${apiKey}`;

            let chatHistory = [];
            chatHistory.push({ role: "user", parts: [{ text: prompt }] });

            const payload = { contents: chatHistory };
            if (isStructured && schema) {
                payload.generationConfig = {
                    responseMimeType: "application/json",
                    responseSchema: schema
                };
            }

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                const result = await response.json();

                if (result.candidates && result.candidates.length > 0 &&
                    result.candidates[0].content && result.candidates[0].content.parts &&
                    result.candidates[0].content.parts.length > 0) {
                    const text = result.candidates[0].content.parts[0].text;
                    if (isStructured) {
                        return JSON.parse(text);
                    } else {
                        return text;
                    }
                } else {
                    console.error('Gemini API Error:', result.error?.message || 'ข้อผิดพลาดที่ไม่ระบุ');
                    return `เกิดข้อผิดพลาด: ${result.error?.message || 'ไม่สามารถสร้างข้อมูลได้'}`;
                }
            } catch (error) {
                console.error('Fetch Error:', error);
                return `เกิดข้อผิดพลาดในการเชื่อมต่อ: ${error.message}`;
            }
        }

        const calculateProgress = (pagesRead, totalPages) => {
            if (totalPages === 0 || !totalPages) return 0;
            return Math.round((pagesRead / totalPages) * 100);
        };

        const renderBooks = () => {
            let filteredBooks = books;
            if (currentView !== 'ทั้งหมด') {
                filteredBooks = books.filter(book => book.status === currentView);
            }

            if (filteredBooks.length === 0) {
                bookListEl.innerHTML = `<p class="text-center text-gray-500 py-8">ไม่พบหนังสือในหมวดนี้</p>`;
                return;
            }

            bookListEl.innerHTML = `
                <table class="w-full text-left table-auto">
                    <thead class="bg-gray-50">
                        <tr>
                            <th class="p-4 font-semibold text-sm">ชื่อหนังสือ</th>
                            <th class="p-4 font-semibold text-sm hidden md:table-cell">ความคืบหน้า</th>
                            <th class="p-4 font-semibold text-sm hidden lg:table-cell">สถานะ</th>
                            <th class="p-4 font-semibold text-sm">เป้าหมายรายวัน (จ-อา)</th>
                        </tr>
                    </thead>
                    <tbody>
                        ${filteredBooks.map(book => {
                            const progress = calculateProgress(book.pagesRead, book.totalPages);
                            const days = ['mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun'];
                            return `
                                <tr class="border-b hover:bg-gray-50 cursor-pointer" onclick="openModal(${book.id})">
                                    <td class="p-4">
                                        <p class="font-bold">${book.name}</p>
                                        <p class="text-sm text-gray-600">${book.author}</p>
                                    </td>
                                    <td class="p-4 hidden md:table-cell">
                                        <div class="flex items-center">
                                            <div class="w-full progress-bar-bg mr-2">
                                                <div class="h-2 progress-bar-fg" style="width: ${progress}%"></div>
                                            </div>
                                            <span class="text-sm font-medium">${progress}%</span>
                                        </div>
                                    </td>
                                    <td class="p-4 hidden lg:table-cell">${book.status}</td>
                                    <td class="p-4">
                                        <div class="flex space-x-1" onclick="event.stopPropagation()">
                                            ${days.map(day => `
                                                <input type="checkbox" id="day-${day}-${book.id}" data-day="${day}" data-id="${book.id}" class="hidden day-checkbox" ${book.dailyGoals[day] ? 'checked' : ''} onchange="updateDailyGoal(this)">
                                                <label for="day-${day}-${book.id}" class="cursor-pointer w-6 h-6 rounded-md border-2 border-gray-300 flex items-center justify-center text-xs font-bold">${day.charAt(0).toUpperCase()}</label>
                                            `).join('')}
                                        </div>
                                    </td>
                                </tr>
                            `;
                        }).join('')}
                    </tbody>
                </table>
            `;
        };
        
        const updateDailyGoal = (checkbox) => {
            const bookId = parseInt(checkbox.dataset.id);
            const day = checkbox.dataset.day;
            const book = books.find(b => b.id === bookId);
            if (book) {
                book.dailyGoals[day] = checkbox.checked;
                saveToLocalStorage();
            }
        };

        const renderDashboard = () => {
            document.getElementById('stat-reading').textContent = books.filter(b => b.status === 'กำลังอ่าน').length;
            const finishedBooks = books.filter(b => b.status === 'อ่านจบแล้ว');
            document.getElementById('stat-finished').textContent = finishedBooks.length;
            document.getElementById('stat-total').textContent = books.length;
            
            const ratedBooks = finishedBooks.filter(b => b.rating);
            const avgRating = ratedBooks.length > 0 ? (ratedBooks.reduce((sum, b) => sum + b.rating, 0) / ratedBooks.length).toFixed(1) : 'N/A';
            document.getElementById('stat-avg-rating').textContent = avgRating;

            renderCategoryChart(finishedBooks);
        };

        const renderCategoryChart = (finishedBooks) => {
            const ctx = document.getElementById('categoryChart').getContext('2d');
            const categoryCounts = finishedBooks.reduce((acc, book) => {
                const cat = book.category || 'ไม่ระบุ';
                acc[cat] = (acc[cat] || 0) + 1;
                return acc;
            }, {});

            const data = {
                labels: Object.keys(categoryCounts),
                datasets: [{
                    data: Object.values(categoryCounts),
                    backgroundColor: ['#8DBFBC', '#F5D491', '#E8A09A', '#B3B3B3', '#C7EAE4', '#F7CAC9'],
                    borderColor: '#FDFBF8',
                    borderWidth: 2,
                    hoverOffset: 4
                }]
            };

            if (categoryChart) {
                categoryChart.destroy();
            }

            categoryChart = new Chart(ctx, {
                type: 'doughnut',
                data: data,
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            position: 'bottom',
                            labels: {
                                font: {
                                    family: "'Sarabun', sans-serif"
                                }
                            }
                        }
                    },
                    cutout: '60%'
                }
            });
        };

        const openModal = (id = null) => {
            bookForm.reset();
            if (id) {
                const book = books.find(b => b.id === id);
                modalTitle.textContent = 'แก้ไขข้อมูลหนังสือ';
                document.getElementById('book-id').value = book.id;
                document.getElementById('name').value = book.name;
                document.getElementById('author').value = book.author;
                document.getElementById('category').value = book.category;
                document.getElementById('totalPages').value = book.totalPages;
                document.getElementById('pagesRead').value = book.pagesRead;
                document.getElementById('status').value = book.status;
                document.getElementById('rating').value = book.rating;
                document.getElementById('startDate').value = book.startDate;
                document.getElementById('endDate').value = book.endDate;
                document.getElementById('summary').value = book.summary;
            } else {
                modalTitle.textContent = 'เพิ่มหนังสือใหม่';
                document.getElementById('book-id').value = '';
                document.getElementById('pagesRead').value = 0;
            }
            modal.classList.remove('hidden');
            // Ensure modalBg and modalContent are available before using them
            if (modalBg && modalContent) {
                setTimeout(() => {
                    modalBg.classList.remove('opacity-0');
                    modalContent.classList.remove('scale-95');
                }, 10);
            }
        };

        const closeModal = () => {
            // Ensure modalBg and modalContent are available before using them
            if (modalBg && modalContent) {
                modalBg.classList.add('opacity-0');
                modalContent.classList.add('scale-95');
                setTimeout(() => modal.classList.add('hidden'), 300);
            } else {
                modal.classList.add('hidden'); // Fallback if for some reason they're null
            }
        };
        
        const saveToLocalStorage = () => {
            localStorage.setItem('books', JSON.stringify(books));
        };

        bookForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const id = document.getElementById('book-id').value;
            const bookData = {
                name: document.getElementById('name').value,
                author: document.getElementById('author').value,
                category: document.getElementById('category').value,
                totalPages: parseInt(document.getElementById('totalPages').value),
                pagesRead: parseInt(document.getElementById('pagesRead').value),
                status: document.getElementById('status').value,
                rating: parseInt(document.getElementById('rating').value) || null,
                startDate: document.getElementById('startDate').value,
                endDate: document.getElementById('endDate').value,
                summary: document.getElementById('summary').value,
            };

            if (id) {
                const bookIndex = books.findIndex(b => b.id == id);
                bookData.id = parseInt(id);
                bookData.dailyGoals = books[bookIndex].dailyGoals;
                books[bookIndex] = bookData;
            } else {
                bookData.id = books.length > 0 ? Math.max(...books.map(b => b.id)) + 1 : 1;
                bookData.dailyGoals = {mon: false, tue: false, wed: false, thu: false, fri: false, sat: false, sun: false};
                books.push(bookData);
            }
            
            saveToLocalStorage();
            renderAll();
            closeModal();
        });

        navButtons.addEventListener('click', (e) => {
            if (e.target.tagName === 'BUTTON') {
                currentView = e.target.dataset.view;
                renderAll();
            }
        });
        
        const updateNavButtons = () => {
             Array.from(navButtons.children).forEach(button => {
                if(button.dataset.view === currentView) {
                    button.classList.add('nav-button-active');
                    button.classList.remove('nav-button-inactive');
                } else {
                    button.classList.add('nav-button-inactive');
                    button.classList.remove('nav-button-active');
                }
            });
        };

        function renderAll() {
            updateNavButtons();
            renderBooks();
            renderDashboard();
        }

        // New function for Summary/Key Takeaways Generator
        async function generateSummary() {
            const bookName = document.getElementById('name').value;
            const author = document.getElementById('author').value;
            const currentSummary = document.getElementById('summary').value;
            const summaryTextarea = document.getElementById('summary');
            const generateSummaryButton = document.getElementById('generate-summary-button');

            if (!bookName) {
                showCustomMessage('ข้อมูลไม่ครบถ้วน', 'กรุณากรอกชื่อหนังสือก่อนครับ');
                return;
            }

            generateSummaryButton.disabled = true;
            summaryTextarea.disabled = true;
            summaryTextarea.value = 'กำลังสร้างสรุป... โปรดรอสักครู่';

            let prompt = `Based on the book titled '${bookName}'`;
            if (author && author.trim() !== '') {
                prompt += ` by '${author}'`;
            }
            if (currentSummary && currentSummary.trim() !== '') {
                prompt += `, and my current brief notes: '${currentSummary.trim()}',`;
            }
            prompt += ` please provide a more detailed summary, key insights, or thought-provoking questions for reflection. Focus on educational or personal growth aspects if applicable, or core plot/themes for fiction. Respond in Thai.`;

            const generatedText = await callGeminiAPI(prompt);

            summaryTextarea.value = generatedText;
            generateSummaryButton.disabled = false;
            summaryTextarea.disabled = false;
        }

        // New function for Weekly Reading Goal Ideas
        async function generateWeeklyGoals() {
            const readingBooksCount = books.filter(b => b.status === 'กำลังอ่าน').length;
            const finishedBooksCount = books.filter(b => b.status === 'อ่านจบแล้ว').length;
            const weeklyGoalsButton = document.getElementById('generate-weekly-goals-button');

            weeklyGoalsButton.disabled = true;
            weeklyGoalsButton.textContent = 'กำลังสร้างเป้าหมาย...';

            let prompt = `ในฐานะโค้ชการอ่าน ผมมีหนังสือที่กำลังอ่านอยู่ ${readingBooksCount} เล่ม และอ่านจบไปแล้ว ${finishedBooksCount} เล่ม โปรดแนะนำ 3 เป้าหมายการอ่านรายสัปดาห์ที่หลากหลายและนำไปปฏิบัติได้จริงสำหรับผม แต่ละเป้าหมายควรสั้นกระชับ (1-2 ประโยค) และให้กำลังใจ ตอบเป็นภาษาไทย`;

            const generatedGoals = await callGeminiAPI(prompt);

            showCustomMessage('ไอเดียเป้าหมายการอ่านรายสัปดาห์ ✨', generatedGoals);
            
            weeklyGoalsButton.disabled = false;
            weeklyGoalsButton.textContent = '✨ ไอเดียเป้าหมายการอ่านรายสัปดาห์';
        }

        // Custom message box function (replace alert)
        function showCustomMessage(title, message) {
            let messageBox = document.getElementById('custom-message-box');
            if (!messageBox) { // Should not happen if HTML is correct, but defensive
                messageBox = document.createElement('div');
                messageBox.id = 'custom-message-box';
                messageBox.className = 'fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-[999] hidden';
                messageBox.innerHTML = `
                    <div class="bg-white rounded-xl shadow-2xl p-6 w-full max-w-sm">
                        <h3 id="message-box-title" class="text-xl font-bold mb-4 text-[#3B6663]"></h3>
                        <p id="message-box-content" class="text-gray-700 mb-6"></p>
                        <div class="text-right">
                            <button onclick="document.getElementById('custom-message-box').classList.add('hidden')" class="bg-[#8DBFBC] hover:bg-[#72a8a5] text-white font-bold py-2 px-4 rounded-lg">ตกลง</button>
                        </div>
                    </div>
                `;
                document.body.appendChild(messageBox);
            }
            document.getElementById('message-box-title').textContent = title;
            document.getElementById('message-box-content').textContent = message;
            messageBox.classList.remove('hidden');
        }


        document.addEventListener('DOMContentLoaded', () => {
            const storedBooks = localStorage.getItem('books');
            if (!storedBooks || JSON.parse(storedBooks).length === 0) {
                books = sampleData;
                localStorage.setItem('books', JSON.stringify(sampleData));
            } else {
                books = JSON.parse(storedBooks);
            }
            
            // Initialize modalBg and modalContent here after DOM is fully loaded
            modalBg = modal.querySelector('.modal-bg');
            modalContent = modal.querySelector('.modal-content');

            renderAll();
        });
        
        modal.addEventListener('click', (e) => {
            if (e.target === modal) {
                closeModal();
            }
        });

    </script>
</body>
</html>
