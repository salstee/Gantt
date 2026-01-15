<!DOCTYPE html>
<html lang="fi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<title>Gantt</title>

<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">

<style>
body {
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
  margin: 0;
  padding: 16px;
  background: #ffffff;
}

h1 {
  margin-top: 0;
}

button {
  width: 100%;
  padding: 16px;
  font-size: 18px;
  margin-bottom: 12px;
  border: none;
  border-radius: 10px;
  background: #007aff;
  color: white;
}

button:active {
  background: #005ecb;
}

.task {
  display: flex;
  align-items: center;
  margin: 8px 0;
}

.task-name {
  width: 110px;
  font-size: 14px;
}

.timeline {
  flex: 1;
  background: #f1f1f1;
  height: 22px;
  border-radius: 6px;
  position: relative;
}

.bar {
  position: absolute;
  height: 22px;
  background: #34c759;
  border-radius: 6px;
}
</style>
</head>

<body>

<h1>Gantt</h1>

<button onclick="startListening()">🎤 Lisää työmaa puhumalla</button>

<div id="gantt"></div>

<script>
let tasks = JSON.parse(localStorage.getItem("tasks")) || [];

function startListening() {
  const SpeechRecognition = window.webkitSpeechRecognition;
  if (!SpeechRecognition) {
    alert("Puheentunnistus ei ole käytettävissä.");
    return;
  }

  const recognition = new SpeechRecognition();
  recognition.lang = "fi-FI";
  recognition.interimResults = false;
  recognition.maxAlternatives = 1;

  recognition.start();

  recognition.onresult = (event) => {
    const text = event.results[0][0].transcript.toLowerCase();
    handleCommand(text);
  };
}

function handleCommand(text) {
  // Esimerkki:
  // "lisää perustukset alkaa 1.3.2026 päättyy 10.3.2026"

  const match = text.match(/lisää (.+?) alkaa (.+?) päättyy (.+)/);
  if (!match) {
    speak("En ymmärtänyt komentoa");
    return;
  }

  const name = match[1];
  const start = parseDate(match[2]);
  const end = parseDate(match[3]);

  if (!start || !end) {
    speak("Päivämäärä ei kelpaa");
    return;
  }

  tasks.push({ name, start, end });
  tasks.sort((a, b) => a.start - b.start);

  localStorage.setItem("tasks", JSON.stringify(tasks));
  speak("Lisäsin työmaan " + name);
  renderGantt();
}

function parseDate(text) {
  if (text.includes("huomenna")) {
    const d = new Date();
    d.setDate(d.getDate() + 1);
    return d;
  }

  const parts = text.match(/\d+/g);
  if (!parts || parts.length < 3) return null;

  return new Date(parts[2], parts[1] - 1, parts[0]);
}

function renderGantt() {
  const gantt = document.getElementById("gantt");
  gantt.innerHTML = "";

  if (tasks.length === 0) return;

  const minDate = Math.min(...tasks.map(t => t.start));
  const maxDate = Math.max(...tasks.map(t => t.end));
  const totalDays = (maxDate - minDate) / 86400000 + 1;

  tasks.forEach(task => {
    const row = document.createElement("div");
    row.className = "task";

    const nameDiv = document.createElement("div");
    nameDiv.className = "task-name";
    nameDiv.textContent = task.name;

    const timeline = document.createElement("div");
    timeline.className = "timeline";

    const bar = document.createElement("div");
    bar.className = "bar";

    const offset = (task.start - minDate) / 86400000;
    const duration = (task.end - task.start) / 86400000 + 1;

    bar.style.left = (offset / totalDays * 100) + "%";
    bar.style.width = (duration / totalDays * 100) + "%";

    timeline.appendChild(bar);
    row.appendChild(nameDiv);
    row.appendChild(timeline);
    gantt.appendChild(row);
  });
}

function speak(text) {
  const utterance = new SpeechSynthesisUtterance(text);
  utterance.lang = "fi-FI";
  speechSynthesis.speak(utterance);
}

renderGantt();
</script>

</body>
</html>