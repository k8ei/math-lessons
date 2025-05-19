<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>تحدي جدول الضرب</title>
  <style>
    body {
      background-color: #ffffff;
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 20px;
      color: #333;
    }
    h1 { margin-bottom: 10px; }
    .level-btn {
      margin: 5px;
      padding: 10px 15px;
      font-size: 16px;
      cursor: pointer;
      border: 1px solid #333;
      background: #000000;
      color: #ffffff;
    }
    .level-btn:hover { background: #222222; }
    #quiz { display: none; margin-top: 20px; }
    #question { font-size: 24px; margin: 20px 0; }
    .options {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 10px;
    }
    .opt-btn {
      padding: 10px 15px;
      font-size: 18px;
      cursor: pointer;
      border: 1px solid #333;
      background: #000000;
      color: #ffffff;
      min-width: 80px;
    }
    .opt-btn:hover { background: #222222; }
    #result, #stats { font-size: 20px; margin-top: 15px; min-height: 24px; }
    #timer { font-size: 20px; color: red; }
    table {
      width: 60%;
      margin: 20px auto;
      border-collapse: collapse;
    }
    th, td {
      border: 1px solid #333;
      padding: 8px;
      text-align: center;
    }
    th {
      background: #000000;
      color: white;
    }
  </style>
</head>
<body>

  <h1>تحدي جدول الضرب</h1>
  <p>أدخل اسمك:</p>
  <input type="text" id="player-name" placeholder="اسمك هنا">
  <p>اختر المستوى:</p>
  <button class="level-btn" onclick="startQuiz(1,5,1)">👍 سهل</button>
  <button class="level-btn" onclick="startQuiz(6,9,2)">💪🏼 متوسط</button>
  <button class="level-btn" onclick="startQuiz(10,15,3)">♛ صعب</button>
  <button class="level-btn" onclick="startChallenge()">🔥 تحدي السرعة</button>

  <div id="quiz">
    <p id="question"></p>
    <p id="timer" style="display: none;"></p> 
    <div class="options" id="options"></div>
    <p id="result"></p>
    <button class="level-btn" onclick="shareScore()">مشاركة نتيجتك</button>
    <p id="stats"></p>
  </div>

  <h2>🏆 جدول الترتيب 🏆</h2>
  <table>
    <thead>
      <tr>
        <th>المركز</th>
        <th>الاسم</th>
        <th>النقاط</th>
      </tr>
    </thead>
    <tbody id="leaderboard"></tbody>
  </table>

  <script>
    let minFactor, maxFactor, correctAnswer, difficulty, usedQuestions = new Set();
    let players = {};  
    let timer;
    let isChallengeMode = false;
    let challengeStats = { total: 0, correct: 0 };
    let challengeTimeLeft = 0;

    function startQuiz(min, max, diff) {
      let playerName = document.getElementById("player-name").value.trim();
      if (!playerName) {
        alert("الرجاء إدخال اسمك أولًا!");
        return;
      }
      minFactor = min;
      maxFactor = max;
      difficulty = diff;
      isChallengeMode = false;
      document.getElementById("quiz").style.display = "block";
      document.getElementById("timer").style.display = "block";
      usedQuestions.clear();
      newQuestion();
    }

    function newQuestion() {
      if (!isChallengeMode) clearInterval(timer);
      let num1, num2, questionKey;
      do {
        num1 = Math.floor(Math.random() * (maxFactor - minFactor + 1)) + minFactor;
        num2 = Math.floor(Math.random() * (maxFactor - minFactor + 1)) + minFactor;
        questionKey = `${num1}-${num2}`;
      } while (usedQuestions.has(questionKey));

      usedQuestions.add(questionKey);
      correctAnswer = num1 * num2;
      document.getElementById("question").innerText = `كم حاصل ضرب ${num1} × ${num2} ؟`;
      document.getElementById("options").innerHTML = generateOptions(correctAnswer);
      document.getElementById("result").innerText = "";

      if (!isChallengeMode) {
        let timeLeft = 15;
        document.getElementById("timer").innerText = `⏳ ${timeLeft} ثانية`;
        timer = setInterval(() => {
          timeLeft--;
          document.getElementById("timer").innerText = `⏳ ${timeLeft} ثانية`;
          if (timeLeft <= 0) {
            clearInterval(timer);
            document.getElementById("result").innerText = "⏳ انتهى الوقت!";
            updateScore(-difficulty);
            setTimeout(newQuestion, 1000);
          }
        }, 1000);
      }
    }

    function generateOptions(correct) {
      let options = new Set([correct]);
      while (options.size < 4) options.add(correct + Math.floor(Math.random() * 10) - 5);
      return Array.from(options).sort(() => Math.random() - 0.5).map(opt =>
        `<button class="opt-btn" onclick="checkAnswer(${opt})">${opt}</button>`
      ).join("");
    }

    function checkAnswer(selected) {
      let playerName = document.getElementById("player-name").value.trim();
      if (!playerName) return;

      if (isChallengeMode) {
        challengeStats.total++;
        if (selected === correctAnswer) {
          challengeStats.correct++;
          document.getElementById("result").innerText = "✅ إجابة صحيحة!";
        } else {
          document.getElementById("result").innerText = "❌ إجابة خاطئة!";
        }
        newQuestion();
      } else {
        clearInterval(timer);
        if (selected === correctAnswer) {
          document.getElementById("result").innerText = "✅ إجابة صحيحة!";
          updateScore(difficulty);
        } else {
          document.getElementById("result").innerText = "❌ إجابة خاطئة!";
          updateScore(-difficulty);
        }
        setTimeout(newQuestion, 1000); // ينتقل تلقائيًا للسؤال التالي بعد ثانية
      }
    }

    function updateScore(points) {
      let playerName = document.getElementById("player-name").value.trim();
      if (!players[playerName]) players[playerName] = 0;
      players[playerName] += points;
      updateLeaderboard();
    }

    function updateLeaderboard() {
      let sortedPlayers = Object.entries(players).sort((a, b) => b[1] - a[1]);
      document.getElementById("leaderboard").innerHTML = sortedPlayers.map(([name, score], i) =>
        `<tr><td>${i + 1}</td><td>${name}</td><td>${score}</td></tr>`
      ).join("");
    }

    function startChallenge() {
      let playerName = document.getElementById("player-name").value.trim();
      if (!playerName) {
        alert("الرجاء إدخال اسمك أولًا!");
        return;
      }
      document.getElementById("quiz").style.display = "block";
      document.getElementById("timer").style.display = "block";
      minFactor = 1;
      maxFactor = 15;
      difficulty = 3;
      usedQuestions.clear();
      isChallengeMode = true;
      challengeStats = { total: 0, correct: 0 };

      challengeTimeLeft = 30;
      document.getElementById("timer").innerText = `⏳ ${challengeTimeLeft} ثانية`;
      newQuestion();

      clearInterval(timer);
      timer = setInterval(() => {
        challengeTimeLeft--;
        document.getElementById("timer").innerText = `⏳ ${challengeTimeLeft} ثانية`;
        if (challengeTimeLeft <= 0) {
          clearInterval(timer);
          document.getElementById("result").innerText = "🏁 انتهى تحدي السرعة!";
          document.getElementById("options").innerHTML = "";
          document.getElementById("stats").innerText =
            `إحصائيات التحدي: أجبت على ${challengeStats.total} أسئلة، ${challengeStats.correct} منها صحيحة.`;
          isChallengeMode = false;
        }
      }, 1000);
    }

    function shareScore() {
      let playerName = document.getElementById("player-name").value.trim();
      let score = players[playerName] || 0;
      let text = `حصلت على ${score} نقاط في تحدي جدول الضرب! هل يمكنك التغلب علي؟`;
      window.open(`https://twitter.com/intent/tweet?text=${encodeURIComponent(text)}`);
    }
  </script>

</body>
</html>
