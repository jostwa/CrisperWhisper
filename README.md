<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Filler Word Detector</title>
<style>
  :root {
    --bg: #0f1115;
    --panel: #171a21;
    --accent: #6c5ce7;
    --accent2: #00d1b2;
    --text: #e8e8ee;
    --muted: #8b8fa3;
    --danger: #ff5470;
    --warn: #ffb020;
  }
  * { box-sizing: border-box; }
  body {
    margin: 0;
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    background: radial-gradient(circle at top, #1c1f29, #0b0c10);
    font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
    color: var(--text);
    padding: 20px;
  }
  .card {
    background: var(--panel);
    border-radius: 16px;
    padding: 32px;
    width: 100%;
    max-width: 560px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.5);
  }
  h1 { font-size: 1.4rem; margin: 0 0 4px; }
  .subtitle { color: var(--muted); font-size: 0.85rem; margin-bottom: 8px; }
  .warning {
    background: #2a2410;
    border: 1px solid #5a4a10;
    color: var(--warn);
    font-size: 0.78rem;
    padding: 10px 12px;
    border-radius: 8px;
    margin-bottom: 20px;
    line-height: 1.4;
  }
  .timer {
    text-align: center;
    font-size: 2.2rem;
    font-variant-numeric: tabular-nums;
    margin-bottom: 16px;
  }
  .controls {
    display: flex;
    gap: 10px;
    justify-content: center;
    margin-bottom: 16px;
  }
  button {
    border: none;
    border-radius: 50px;
    padding: 12px 20px;
    font-size: 0.95rem;
    font-weight: 600;
    cursor: pointer;
    transition: transform .15s ease, opacity .15s ease;
  }
  button:hover { transform: translateY(-2px); }
  button:disabled { opacity: 0.35; cursor: not-allowed; transform: none; }
  .btn-record { background: var(--danger); color: white; }
  .btn-stop { background: #444a5c; color: white; }
  .status {
    text-align: center;
    color: var(--muted);
    font-size: 0.85rem;
    margin-bottom: 20px;
    min-height: 18px;
  }
  .stats-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
    margin-bottom: 20px;
  }
  .stat-box {
    background: #1e222c;
    border-radius: 10px;
    padding: 12px;
    text-align: center;
  }
  .stat-box .num {
    font-size: 1.6rem;
    font-weight: 700;
    color: var(--accent2);
  }
  .stat-box .label {
    font-size: 0.7rem;
    color: var(--muted);
    text-transform: uppercase;
    letter-spacing: 0.5px;
    margin-top: 4px;
  }
  .section-title {
    font-size: 0.9rem;
    color: var(--muted);
    text-transform: uppercase;
    letter-spacing: 1px;
    margin: 0 0 10px;
    border-top: 1px solid #262a35;
    padding-top: 16px;
  }
  .transcript {
    background: #0b0c10;
    border-radius: 10px;
    padding: 12px;
    font-size: 0.9rem;
    line-height: 1.6;
    max-height: 140px;
    overflow-y: auto;
    margin-bottom: 16px;
    color: var(--text);
  }
  .transcript mark {
    background: var(--danger);
    color: white;
    border-radius: 4px;
    padding: 0 4px;
  }
  .filler-log {
    max-height: 220px;
    overflow-y: auto;
  }
  .filler-row {
    display: flex;
    justify-content: space-between;
    background: #1e222c;
    border-radius: 8px;
    padding: 8px 12px;
    margin-bottom: 6px;
    font-size: 0.85rem;
  }
  .filler-row .word {
    color: var(--danger);
    font-weight: 600;
  }
  .filler-row .when {
    color: var(--muted);
    font-variant-numeric: tabular-nums;
  }
  .empty {
    color: var(--muted);
    font-size: 0.85rem;
    text-align: center;
    padding: 16px 0;
  }
  .pulse {
    display: inline-block;
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background: var(--danger);
    margin-right: 6px;
    animation: pulse 1s infinite;
  }
  @keyframes pulse { 0%,100% {opacity:1;} 50% {opacity:0.2;} }
  .download-row {
    display: flex;
    justify-content: center;
    margin-top: 16px;
  }
  .btn-export {
    background: var(--accent);
    color: white;
  }
</style>
</head>
<body>

<div class="card">
  <h1>🎙️ Filler Word Detector</h1>
  <div class="subtitle">Records audio and flags speech fillers like "um", "uh", "hmm", "erm" as they're detected, with timestamps.</div>
  <div class="warning" id="browserWarning">
    Note: this relies on your browser's built-in speech recognition (Chrome/Edge only). Speech engines are built to produce clean text, so they don't always transcribe filler sounds — treat counts as an estimate, not an exact measurement.
  </div>

  <div class="timer" id="timer">00:00</div>

  <div class="controls">
    <button class="btn-record" id="recordBtn">● Start Recording</button>
    <button class="btn-stop" id="stopBtn" disabled>⏹ Stop</button>
  </div>

  <div class="status" id="status">Click "Start Recording" to begin.</div>

  <div class="stats-grid">
    <div class="stat-box">
      <div class="num" id="fillerCount">0</div>
      <div class="label">Fillers</div>
    </div>
    <div class="stat-box">
      <div class="num" id="fillerTime">0.0s</div>
      <div class="label">Filler Time</div>
    </div>
    <div class="stat-box">
      <div class="num" id="fillerRate">0.0</div>
      <div class="label">Per Minute</div>
    </div>
  </div>

  <div class="section-title">Live Transcript</div>
  <div class="transcript" id="transcript"><span style="color:var(--muted)">Transcript will appear here...</span></div>

  <div class="section-title">Filler Log</div>
  <div class="filler-log" id="fillerLog">
    <div class="empty" id="emptyLog">No fillers detected yet.</div>
  </div>

  <div class="download-row">
    <button class="btn-export" id="exportBtn" disabled>⬇ Export Report (JSON)</button>
  </div>
</div>

<script>
const FILLER_PATTERNS = [
  'um', 'umm', 'ummm', 'uh', 'uhh', 'uhm', 'erm', 'ehm', 'hmm', 'hmmm',
  'ah', 'ahh', 'er', 'err', 'like', 'you know', 'i mean', 'so yeah'
];
// Core non-lexicalized fillers we score by default (exclude filler *phrases* like "like"/"you know" from strict count,
// but still highlight them). Toggle via CORE_ONLY below if you want stricter detection.
const CORE_FILLERS = new Set(['um','umm','ummm','uh','uhh','uhm','erm','ehm','hmm','hmmm','ah','ahh','er','err']);

const recordBtn = document.getElementById('recordBtn');
const stopBtn = document.getElementById('stopBtn');
const status = document.getElementById('status');
const timerEl = document.getElementById('timer');
const transcriptEl = document.getElementById('transcript');
const fillerLog = document.getElementById('fillerLog');
const emptyLog = document.getElementById('emptyLog');
const fillerCountEl = document.getElementById('fillerCount');
const fillerTimeEl = document.getElementById('fillerTime');
const fillerRateEl = document.getElementById('fillerRate');
const exportBtn = document.getElementById('exportBtn');
const browserWarning = document.getElementById('browserWarning');

let recognition;
let mediaRecorder;
let audioChunks = [];
let stream;
let startTime;
let timerInterval;
let fillers = [];
let fullTranscript = '';
let estFillerDuration = 0.4; // seconds, rough estimate per filler utterance

const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
if (!SpeechRecognition) {
  browserWarning.textContent = 'Your browser does not support the Web Speech API. Please use Chrome or Edge for filler detection (audio recording will still work, but no transcript/filler detection).';
}

function formatTime(ms) {
  const totalSec = Math.floor(ms / 1000);
  const min = String(Math.floor(totalSec / 60)).padStart(2, '0');
  const sec = String(totalSec % 60).padStart(2, '0');
  return `${min}:${sec}`;
}

function updateTimer() {
  timerEl.textContent = formatTime(Date.now() - startTime);
}

function updateStats() {
  const count = fillers.length;
  const totalTime = fillers.reduce((sum, f) => sum + f.duration, 0);
  const elapsedMin = Math.max((Date.now() - startTime) / 60000, 1/60);
  fillerCountEl.textContent = count;
  fillerTimeEl.textContent = totalTime.toFixed(1) + 's';
  fillerRateEl.textContent = (count / elapsedMin).toFixed(1);
}

function logFiller(word, timestampMs) {
  const relativeSec = ((timestampMs - startTime) / 1000).toFixed(1);
  fillers.push({ word, timestampMs, relativeSec: parseFloat(relativeSec), duration: estFillerDuration });

  emptyLog.style.display = 'none';
  const row = document.createElement('div');
  row.className = 'filler-row';
  row.innerHTML = `<span class="word">"${word}"</span><span class="when">${relativeSec}s in</span>`;
  fillerLog.prepend(row);

  updateStats();
}

function scanTextForFillers(text, timestampMs) {
  const words = text.toLowerCase().replace(/[.,!?;:]/g, '').split(/\s+/);
  words.forEach(w => {
    if (CORE_FILLERS.has(w)) {
      logFiller(w, timestampMs);
    }
  });
}

function highlightTranscript(text) {
  let html = text;
  CORE_FILLERS.forEach(f => {
    const re = new RegExp(`\\b${f}\\b`, 'gi');
    html = html.replace(re, m => `<mark>${m}</mark>`);
  });
  return html;
}

async function startRecording() {
  try {
    stream = await navigator.mediaDevices.getUserMedia({ audio: true });
  } catch (err) {
    status.textContent = 'Microphone access denied or unavailable.';
    return;
  }

  audioChunks = [];
  mediaRecorder = new MediaRecorder(stream);
  mediaRecorder.ondataavailable = e => { if (e.data.size > 0) audioChunks.push(e.data); };
  mediaRecorder.onstop = () => {
    stream.getTracks().forEach(track => track.stop());
  };
  mediaRecorder.start();

  fillers = [];
  fullTranscript = '';
  fillerLog.innerHTML = '';
  emptyLog.style.display = 'block';
  emptyLog.textContent = 'No fillers detected yet.';
  fillerLog.appendChild(emptyLog);
  transcriptEl.innerHTML = '<span style="color:var(--muted)">Listening...</span>';

  startTime = Date.now();
  timerInterval = setInterval(updateTimer, 200);

  if (SpeechRecognition) {
    recognition = new SpeechRecognition();
    recognition.continuous = true;
    recognition.interimResults = true;
    recognition.lang = 'en-US';

    recognition.onresult = (event) => {
      let interim = '';
      for (let i = event.resultIndex; i < event.results.length; i++) {
        const result = event.results[i];
        const text = result[0].transcript;
        const now = Date.now();
        if (result.isFinal) {
          fullTranscript += text + ' ';
          scanTextForFillers(text, now);
        } else {
          interim += text;
          scanTextForFillers(text, now);
        }
      }
      transcriptEl.innerHTML = highlightTranscript(fullTranscript) + '<span style="color:var(--muted)">' + interim + '</span>';
    };

    recognition.onerror = (e) => {
      if (e.error !== 'no-speech') {
        status.textContent = 'Speech recognition error: ' + e.error;
      }
    };

    recognition.onend = () => {
      if (mediaRecorder && mediaRecorder.state === 'recording') {
        recognition.start(); // auto-restart if still recording (some browsers stop after silence)
      }
    };

    recognition.start();
  }

  status.innerHTML = '<span class="pulse"></span>Recording & listening...';
  recordBtn.disabled = true;
  stopBtn.disabled = false;
  exportBtn.disabled = true;
}

function stopRecording() {
  clearInterval(timerInterval);
  mediaRecorder.stop();
  if (recognition) {
    recognition.onend = null; // prevent auto-restart
    recognition.stop();
  }
  recordBtn.disabled = false;
  stopBtn.disabled = true;
  exportBtn.disabled = fillers.length === 0 && fullTranscript.trim() === '';
  status.textContent = `Done. ${fillers.length} filler(s) detected in ${timerEl.textContent}.`;
  updateStats();
}

exportBtn.addEventListener('click', () => {
  const report = {
    recordedAt: new Date().toISOString(),
    durationLabel: timerEl.textContent,
    transcript: fullTranscript.trim(),
    fillerCount: fillers.length,
    totalFillerTimeSec: fillers.reduce((s, f) => s + f.duration, 0),
    fillers: fillers.map(f => ({ word: f.word, atSecond: f.relativeSec, estDurationSec: f.duration }))
  };
  const blob = new Blob([JSON.stringify(report, null, 2)], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `filler-report-${Date.now()}.json`;
  a.click();
  URL.revokeObjectURL(url);
});

recordBtn.addEventListener('click', startRecording);
stopBtn.addEventListener('click', stopRecording);
</script>

</body>
</html>
