# task-organizer
انجز مهامك مع الحماس والتحفيز 
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>📋 جدول تنظيم المهام</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    .dark-mode { background-color: #1a202c; color: white; }
    .dark-mode input, .dark-mode table, .dark-mode textarea {
      background-color: #2d3748;
      color: white;
      border-color: #4a5568;
    }

    /* تنسيق النجمة */
    #star {
      font-size: 60px;
      color: gold;
      text-align: center;
      display: none;
    }

    .motivationalText {
      text-align: center;
      font-size: 24px;
      color: #4CAF50;
      font-weight: bold;
      display: none;
    }

    /* تصميم الزر "تم الإنجاز" */
    .doneButton {
      background-color: #4CAF50;
      color: white;
      padding: 10px 20px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }

    .doneButton:hover {
      background-color: #45a049;
    }
  </style>
</head>
<body class="bg-gray-100 min-h-screen p-6 transition-all duration-500" id="mainBody">

  <div class="max-w-5xl mx-auto bg-white rounded-xl shadow p-6 space-y-6 dark:bg-gray-800 dark:text-white">
    <div class="flex justify-between items-center">
      <h1 class="text-3xl font-bold text-blue-600 dark:text-blue-300">📅 جدول تنظيم المهام</h1>
      <button onclick="toggleDarkMode()" class="text-sm bg-gray-200 hover:bg-gray-300 px-4 py-1 rounded dark:bg-gray-600 dark:text-white">🌓 الوضع الليلي</button>
    </div>

    <p id="motivationalText" class="motivationalText">شطووور! 💪</p>

    <!-- عرض النجمة عند الإنجاز -->
    <div id="star">⭐</div>

    <!-- شريط التقدم -->
    <div class="w-full bg-gray-200 rounded-full h-4">
      <div id="progressBar" class="bg-green-500 h-4 rounded-full transition-all duration-700" style="width: 0%"></div>
    </div>

    <!-- الإحصائيات -->
    <div class="text-center font-semibold text-gray-700 dark:text-gray-200">
      <span>✅ المهام المنجزة: <span id="doneCount">0</span></span> / <span>📋 الإجمالي: <span id="totalCount">0</span></span>
    </div>

    <!-- البحث -->
    <input type="text" id="searchInput" onkeyup="filterTasks()" placeholder="🔍 ابحث عن مهمة..." class="w-full p-2 border rounded mt-2 dark:bg-gray-700 dark:border-gray-500" />

    <!-- إدخال المهام -->
    <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
      <input id="day" placeholder="اليوم" class="p-2 border rounded dark:bg-gray-700 dark:border-gray-500" />
      <input id="task" placeholder="المهمة" class="p-2 border rounded dark:bg-gray-700 dark:border-gray-500" />
      <input id="time" placeholder="الوقت" class="p-2 border rounded dark:bg-gray-700 dark:border-gray-500" />
      <input id="note" placeholder="ملاحظات" class="p-2 border rounded dark:bg-gray-700 dark:border-gray-500" />
    </div>
    <button onclick="addTask()" class="bg-blue-600 hover:bg-blue-700 text-white px-6 py-2 rounded">➕ إضافة</button>

    <!-- جدول المهام -->
    <div class="overflow-x-auto">
      <table class="w-full text-right border mt-4">
        <thead class="bg-blue-100 text-blue-800 dark:bg-blue-900">
          <tr>
            <th class="border p-2">اليوم</th>
            <th class="border p-2">المهمة</th>
            <th class="border p-2">الوقت</th>
            <th class="border p-2">ملاحظات</th>
            <th class="border p-2">إجراء</th>
          </tr>
        </thead>
        <tbody id="taskTable" class="bg-white dark:bg-gray-700 text-gray-700 dark:text-white"></tbody>
      </table>
    </div>
  </div>

  <!-- صوت النغمة -->
  <audio id="successSound" src="https://cdn.pixabay.com/audio/2022/03/15/audio_59ca755cd4.mp3"></audio>

  <!-- JavaScript -->
  <script>
    let tasks = JSON.parse(localStorage.getItem('tasks')) || [];
    const messages = [
      "أنت تصنع فرقًا! 💪",
      "استمر، النجاح قريب! 🌟",
      "إنجاز وراء إنجاز! 🚀",
      "أحسنت، عمل رائع! 👏",
      "خطوة نحو أهدافك! 🏆"
    ];

    function renderTasks() {
      const table = document.getElementById('taskTable');
      table.innerHTML = '';

      const searchValue = document.getElementById('searchInput').value.toLowerCase();
      let done = 0;

      tasks.forEach((task, index) => {
        if (
          task.day.toLowerCase().includes(searchValue) ||
          task.task.toLowerCase().includes(searchValue) ||
          task.note.toLowerCase().includes(searchValue)
        ) {
          const row = `
            <tr class="${task.done ? 'bg-green-100 dark:bg-green-700' : ''}">
              <td class="border p-2">${task.day}</td>
              <td class="border p-2">${task.task}</td>
              <td class="border p-2">${task.time}</td>
              <td class="border p-2">${task.note}</td>
              <td class="border p-2 text-center space-y-1">
                <button onclick="markAsDone(${index})" class="doneButton">✅ تم الإنجاز</button>
                <button onclick="deleteTask(${index})" class="text-red-600 hover:text-red-800 block">🗑️ حذف</button>
              </td>
            </tr>
          `;
          table.innerHTML += row;
        }

        if (task.done) done++;
      });

      document.getElementById('doneCount').textContent = done;
      document.getElementById('totalCount').textContent = tasks.length;
      document.getElementById('progressBar').style.width = `${(done / tasks.length) * 100 || 0}%`;

      showMotivationalText();
    }

    function addTask() {
      const day = document.getElementById('day').value.trim();
      const task = document.getElementById('task').value.trim();
      const time = document.getElementById('time').value.trim();
      const note = document.getElementById('note').value.trim();

      if (!day || !task || !time) {
        alert('يرجى ملء جميع البيانات');
        return;
      }

      tasks.push({ day, task, time, note, done: false });
      localStorage.setItem('tasks', JSON.stringify(tasks));
      renderTasks();

      document.getElementById('day').value = '';
      document.getElementById('task').value = '';
      document.getElementById('time').value = '';
      document.getElementById('note').value = '';
    }

    function deleteTask(index) {
      tasks.splice(index, 1);
      localStorage.setItem('tasks', JSON.stringify(tasks));
      renderTasks();
    }

    function markAsDone(index) {
      tasks[index].done = true;
      localStorage.setItem('tasks', JSON.stringify(tasks));
      renderTasks();
      playSuccessSound();
      document.getElementById('star').style.display = 'block'; // عرض النجمة
      document.getElementById('motivationalText').style.display = 'block'; // عرض الكلمة التحفيزية
    }

    function playSuccessSound() {
      const audio = document.getElementById('successSound');
      audio.play();
    }

    function toggleDarkMode() {
      document.getElementById('mainBody').classList.toggle('dark-mode');
    }

    function showMotivationalText() {
      const text = messages[Math.floor(Math.random() * messages.length)];
      document.getElementById('motivationalText').textContent = text;
    }

    renderTasks();
  </script>
</body>
</html>
