# LearnERA
Welcome to LearnERA ! This is a simple , browser based AI assistant I designed to help students with their daily academic challnges.
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LearnERA - Student AI Agent</title>
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'%3E%3Cpath fill='%236c63ff' d='M50,5 C74.85,5 95,25.15 95,50 C95,74.85 74.85,95 50,95 C25.15,95 5,74.85 5,50 C5,25.15 25.15,5 50,5 Z M50,15 C30.67,15 15,30.67 15,50 C15,69.33 30.67,85 50,85 C69.33,85 85,69.33 85,50 C85,30.67 69.33,15 50,15 Z M50,40 C55.52,40 60,44.48 60,50 C60,55.52 55.52,60 50,60 C44.48,60 40,55.52 40,50 C40,44.48 44.48,40 50,40 Z'%3E%3C/path%3E%3C/svg%3E">
<style>
  :root { --primary-color: #6c63ff; }
  body { background-color: #121212; color: #f5f5f5; font-family: 'Segoe UI', sans-serif; display: flex; justify-content: center; align-items: flex-start; min-height: 100vh; padding-top: 50px; padding-bottom: 50px; }
  .container { background-color: #1e1e1e; border-radius: 15px; padding: 30px; width: 90%; max-width: 500px; box-shadow: 0 0 20px rgba(0,0,0,0.6); position: relative; }
  h1 { text-align: center; color: var(--primary-color); }
  select, input, textarea, button { width: 100%; margin: 10px 0; padding: 12px; border-radius: 8px; border: none; font-size: 16px; box-sizing: border-box; }
  select, input, textarea { background-color: #2c2c2c; color: #f5f5f5; border: 1px solid #444; }
  select:focus, input:focus, textarea:focus { outline: none; border-color: var(--primary-color); }
  button { background-color: var(--primary-color); color: white; font-weight: bold; cursor: pointer; transition: background 0.3s; }
  button:hover { background-color: #5b52d1; }
  button:disabled { background-color: #555; cursor: not-allowed; }
  .response { background-color: #2c2c2c; padding: 15px; border-radius: 10px; margin-top: 15px; min-height: 80px; white-space: pre-wrap; word-wrap: break-word; }
  .modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0, 0, 0, 0.7); display: flex; justify-content: center; align-items: center; z-index: 1000; transition: opacity 0.3s; }
  .modal-hidden { display: none; }
  .modal-content { background: #1e1e1e; padding: 25px; border-radius: 15px; text-align: center; width: 90%; max-width: 450px; border: 1px solid #444; }
  .modal-content h2 { color: var(--primary-color); }
  .modal-content p { font-size: 14px; color: #ccc; margin-bottom: 5px; }
  .modal-content input { margin-top: 15px; margin-bottom: 15px; }
  .modal-content a { font-size: 14px; color: var(--primary-color); text-decoration: underline; }
  .modal-content details { border: 1px solid #444; border-radius: 8px; padding: 10px; margin-top: 15px; text-align: left; background-color: #2c2c2c; }
  .modal-content summary { cursor: pointer; font-weight: bold; color: #f5f5f5; }
  .modal-content ol { padding-left: 25px; font-size: 14px; color: #ccc; line-height: 1.6; }
  .modal-content li { margin-bottom: 8px; }
  .reset-key { font-size: 12px; background: none; color: #aaa; text-decoration: underline; cursor: pointer; padding: 5px; margin-top: 15px; }
  .footer-credit { text-align: center; font-size: 12px; color: #888; margin-top: 25px; }
</style>
</head>
<body>

<div id="apiKeyModal" class="modal-overlay modal-hidden">
  <div class="modal-content">
    <h2>Welcome to LearnERA!</h2>
    <p>Please enter your Google AI API key to begin.</p>
    <a href="https://aistudio.google.com/app/apikey" target="_blank">Get your free API key here.</a>
    <details>
      <summary>How do I get an API key?</summary>
      <ol>
        <li>Click the link above to open Google AI Studio.</li>
        <li>Sign in with your Google Account.</li>
        <li>Click <strong>+ Create API key</strong>.</li>
        <li>Copy your secret key and paste it below.</li>
      </ol>
    </details>
    <input type="password" id="apiKeyInput" placeholder="Enter your Gemini API Key...">
    <button id="saveApiKeyBtn">Save & Start</button>
  </div>
</div>

<div class="container">
  <h1>LearnERA</h1>
  <select id="feature">
    <option value="doubt">AI Doubt Solver</option>
    <option value="notes">Notes & Summarization</option>
    <option value="planner">Task Planner & Reminders</option>
    <option value="quiz">Quiz & Self-Assessment</option>
    <option value="motivation">Motivation & Productivity Booster</option>
  </select>

  <textarea id="userInput" rows="4" placeholder="Type your question, notes, or request here..."></textarea>
  <button id="submitBtn">Send to LearnERA</button>

  <div class="response" id="responseArea">LearnERA is ready to help! 🌌</div>
  <button id="resetKeyBtn" class="reset-key">Change API Key</button>

  <p class="footer-credit">Created with love and care by team ByteBOND ❤️</p>
</div>

<script type="module">
  // Using the latest Google Gen AI SDK for JS
  import { GoogleGenerativeAI } from "https://esm.sh/@google/generative-ai@latest";

  const submitBtn = document.getElementById('submitBtn');
  const userInput = document.getElementById('userInput');
  const featureSelect = document.getElementById('feature');
  const responseArea = document.getElementById('responseArea');
  const apiKeyModal = document.getElementById('apiKeyModal');
  const apiKeyInput = document.getElementById('apiKeyInput');
  const saveApiKeyBtn = document.getElementById('saveApiKeyBtn');
  const resetKeyBtn = document.getElementById('resetKeyBtn');

  let genAI;

  function initializeAI(apiKey) {
    if (!apiKey) return;
    try {
      genAI = new GoogleGenerativeAI(apiKey);
      apiKeyModal.classList.add('modal-hidden');
    } catch (err) {
      alert("Error initializing AI. Check your key.");
      localStorage.removeItem('geminiApiKey');
      apiKeyModal.classList.remove('modal-hidden');
    }
  }

  window.addEventListener('load', () => {
    const savedApiKey = localStorage.getItem('geminiApiKey');
    if (savedApiKey) initializeAI(savedApiKey);
    else apiKeyModal.classList.remove('modal-hidden');
  });

  saveApiKeyBtn?.addEventListener('click', () => {
    const apiKey = apiKeyInput.value.trim();
    if (apiKey) {
      localStorage.setItem('geminiApiKey', apiKey);
      initializeAI(apiKey);
    }
  });

  resetKeyBtn?.addEventListener('click', () => {
    localStorage.removeItem('geminiApiKey');
    genAI = null; 
    apiKeyModal.classList.remove('modal-hidden');
  });

  function getSystemPrompt(feature) {
    switch (feature) {
      case 'doubt': return "You are an AI Doubt Solver. Clearly and concisely answer the student's question.";
      case 'notes': return "You are a note-taking assistant. Summarize the following text into key bullet points.";
      case 'planner': return "You are a task planner. Create a simple, actionable to-do list.";
      case 'quiz': return "You are a quiz master. Generate a 3-question MCQ with answers at the end.";
      case 'motivation': return "You are a motivational coach. Provide an encouraging message.";
      default: return "You are a helpful student assistant.";
    }
  }

  async function callAI() {
    if (!genAI) return;
    const input = userInput.value.trim();
    if (!input) return;

    submitBtn.disabled = true;
    submitBtn.textContent = "Thinking...";
    responseArea.textContent = "LearnERA is thinking… 🤔";

    try {
      // UPDATED: Using gemini-3-flash-preview for latest 2026 capabilities
      const model = genAI.getGenerativeModel({ 
        model: "gemini-3-flash-preview",
        systemInstruction: getSystemPrompt(featureSelect.value)
      });

      const result = await model.generateContent(input);
      const response = await result.response;
      responseArea.textContent = response.text();
    } catch (err) {
      responseArea.textContent = "⚠️ Connection error. Please verify your API key or try again later.";
      console.error(err);
    } finally {
      submitBtn.disabled = false;
      submitBtn.textContent = "Send to LearnERA";
    }
  }

  submitBtn?.addEventListener('click', callAI);
</script>
</body>
</html>