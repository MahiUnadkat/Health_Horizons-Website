<body>
  <h1>Voice-Enabled Medicine & Water Reminder</h1>

  <!-- Frame 1: Set Reminder -->
  <div class="frame">
    <h2>Set New Reminder</h2>
    <input type="time" id="reminderTime" />
    <input type="date" id="reminderDate" />
    <input type="text" id="reminderMessage" placeholder="Enter reminder message" />
    <label>Repeat:</label>
    <select id="repeatSelect">
      <option value="none">No Repeat</option>
      <option value="daily">Daily</option>
      <option value="weekdays">Weekdays</option>
      <option value="weekends">Weekends</option>
    </select>
    <button onclick="addReminder()">Add Reminder</button>
  </div>

  <!-- Frame 2: Voice Input -->
  <div class="frame">
    <h2>Voice Command</h2>
    <button onclick="startVoiceInput()">🎤 Start Voice Input</button>
    <div id="voiceOutput">Say "Remind me to take medicine at 10:30 on 2023-05-01"</div>
  </div>

  <!-- Frame 3: Reminders List -->
  <div class="frame">
    <h2>Upcoming Reminders</h2>
    <div id="reminderList" class="reminder-list"></div>
  </div>

  <!-- Frame 4: Alert -->
  <div class="frame">
    <h2>Upcoming Alert</h2>
    <div id="alert" class="alert">No current alert</div>
  </div>

  <!-- Beep Sound -->
  <audio id="alarmSound" src="https://www.soundjay.com/buttons/sounds/beep-07.mp3" preload="auto"></audio>

  <script>
    let reminders = JSON.parse(localStorage.getItem('reminders')) || [];
    let lastAlertTime = "";

    function saveReminders() {
      localStorage.setItem('reminders', JSON.stringify(reminders));
    }

    function addReminder() {
      const time = document.getElementById('reminderTime').value;
      const date = document.getElementById('reminderDate').value;
      const message = document.getElementById('reminderMessage').value.trim();
      const repeat = document.getElementById('repeatSelect').value;

      if (!time || !date || !message) {
        alert("Please enter all details.");
        return;
      }

      const reminder = {
        time,
        date,
        message,
        repeat
      };

      reminders.push(reminder);
      saveReminders();
      displayReminders();
      document.getElementById('reminderTime').value = "";
      document.getElementById('reminderDate').value = "";
      document.getElementById('reminderMessage').value = "";
    }

    function deleteReminder(index) {
      reminders.splice(index, 1);
      saveReminders();
      displayReminders();
    }

    function displayReminders() {
      const list = document.getElementById('reminderList');
      list.innerHTML = "";
      reminders.forEach((r, i) => {
        const div = document.createElement("div");
        div.className = "reminder-item";
        div.innerHTML = `<span>${r.time} - ${r.message} (Repeat: ${r.repeat}, Date: ${r.date})</span>
                         <button class="delete-btn" onclick="deleteReminder(${i})">Delete</button>`;
        list.appendChild(div);
      });
    }

    function checkReminders() {
      const now = new Date();
      const currentTime = now.toTimeString().slice(0, 5); // HH:MM
      const currentDate = now.toISOString().split('T')[0]; // YYYY-MM-DD
      const alertDiv = document.getElementById("alert");
      const found = reminders.find(r => r.time === currentTime && r.date === currentDate);

      if (found && lastAlertTime !== currentTime) {
        alertDiv.textContent = `🔔 ${found.message}`;
        document.getElementById('alarmSound').play();
        showNotification(found.message);
        lastAlertTime = currentTime;
      } else if (!found && alertDiv.textContent !== "No current alert") {
        alertDiv.textContent = "No current alert";
      }
    }

    function showNotification(message) {
      if (Notification.permission === 'granted') {
        new Notification("⏰ Reminder", {
          body: message,
          icon: "https://cdn-icons-png.flaticon.com/512/565/565547.png"
        });
      }
    }

    function startVoiceInput() {
      if (!('webkitSpeechRecognition' in window || 'SpeechRecognition' in window)) {
        alert("Your browser doesn't support voice recognition.");
        return;
      }

      const recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
      recognition.lang = 'en-US';
      recognition.interimResults = false;
      recognition.maxAlternatives = 1;

      recognition.start();
      document.getElementById("voiceOutput").textContent = "Listening...";

      recognition.onresult = function(event) {
        const text = event.results[0][0].transcript;
        document.getElementById("voiceOutput").textContent = `You said: "${text}"`;

        const timeMatch = text.match(/\b\d{1,2}:\d{2}\b/);
        const dateMatch = text.match(/\d{4}-\d{2}-\d{2}/);
        const msgMatch = text.match(/remind me to (.+?) at/i);

        if (timeMatch && dateMatch && msgMatch) {
          const time = timeMatch[0];
          const date = dateMatch[0];
          const message = msgMatch[1];
          reminders.push({ time, date, message, repeat: 'none' });
          saveReminders();
          displayReminders();
        } else {
          document.getElementById("voiceOutput").textContent += " (Couldn't parse reminder)";
        }
      };

      recognition.onerror = function(event) {
        document.getElementById("voiceOutput").textContent = "Error: " + event.error;
      };
    }

    // Request notification permission if not granted
    if (Notification.permission !== 'granted' && Notification.permission !== 'denied') {
      Notification.requestPermission();
    }

    displayReminders();
    setInterval(checkReminders, 1000);
  </script>
</body>
</html>
