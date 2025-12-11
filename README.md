<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Block Game with Achievements</title>
<style>
body {
    font-family: "Segoe UI", Arial, sans-serif;
    background: linear-gradient(150deg, #0f172a, #1e293b);
    color: white;
    text-align: center;
    padding: 20px;
}
h1 { font-size: 38px; margin-bottom: 5px; }
#level { font-size: 22px; color: #38bdf8; margin-bottom: 10px; }

.selectLang {
    padding: 10px 15px;
    font-size: 18px;
    margin-bottom: 10px;
    border-radius: 6px;
    border: none;
    cursor: pointer;
    background: rgba(255,255,255,0.15);
    color: white;
    appearance: none;
    -webkit-appearance: none;
    -moz-appearance: none;
    text-align: center;
}
.selectLang:hover { background: rgba(0,0,0,0.3); }

.btn {
    padding: 12px 22px; margin: 6px; background: rgba(255,255,255,0.15);
    border: none; border-radius: 8px; color: white; font-size: 18px; cursor: pointer;
    transition: 0.2s;
}
.btn:hover { background: rgba(0,0,0,0.3); }
.btn:active { background: rgba(0,0,0,0.5); transform: scale(0.9); }

#task { font-size: 28px; margin-top: 20px; min-height: 60px; }
#answer { margin-top: 15px; font-size: 24px; padding: 10px; width: 160px; border-radius: 6px; border: none; text-align: center; }
#result { margin-top: 15px; font-size: 24px; min-height: 40px; }

#blocks { margin-top: 25px; display: flex; justify-content: center; gap: 8px; flex-wrap: wrap; min-height: 90px; }
.block { width: 45px; height: 45px; background: #38bdf8; border-radius: 8px; }

#xp-container { margin-top:20px; width:80%; background:#1f2937; height:24px; border-radius:12px; margin-left:auto;margin-right:auto; }
#xp-bar { height:100%; width:0%; background:#10b981; border-radius:12px; transition:width 0.5s; }

#achievements {
    margin-top: 30px;
    text-align: left;
    width: 60%;
    margin-left: auto;
    margin-right: auto;
    background: rgba(255,255,255,0.05);
    padding: 15px;
    border-radius: 10px;
}
#achievements h3 { color: #facc15; }
.achievement { margin: 6px 0; color: #cbd5e1; }
.achievement.unlocked { color: #10b981; font-weight: bold; }

#footer { margin-top:40px; font-size:18px; color:#94a3b8; }
</style>
</head>
<body>

<h1 id="title">Block Game</h1>
<div id="level">Level: 1</div>

<!-- Språkvelger -->
<select class="selectLang" id="languageSelect" onchange="setLanguage(this.value)">
    <option value="no">Norsk</option>
    <option value="en" selected>English</option>
    <option value="uk">Українська</option>
    <option value="ru">Русский</option>
    <option value="ar">العربية</option>
    <option value="sv">Svenska</option>
    <option value="pl">Polski</option>
    <option value="da">Dansk</option>
    <option value="ur">اردو</option>
</select>

<h3 id="difficultyTitle">Difficulty:</h3>
<div id="difficultyButtons"></div>

<div id="task">Select difficulty to start…</div>

<input id="answer" type="number" placeholder="Type number of blocks" oninput="checkAnswer()">
<div id="result"></div>

<div id="blocks"></div>

<div id="xp-container">
    <div id="xp-bar"></div>
</div>

<div id="achievements">
    <h3>Achievements</h3>
    <div class="achievement" id="ach-level5">Reach Level 5</div>
    <div class="achievement" id="ach-level10">Reach Level 10</div>
    <div class="achievement" id="ach-100xp">Earn 100 XP</div>
</div>

<div id="footer">Made in Norway</div>

<script>
let lang = "en";
let difficulty = null;
let currentAnswer = 0;
let level = 1;
let xp = 0;
let xpToLevel = 10;
let totalXP = 0;

// Tekster
const text = {
    no:{title:"Klosse-spill", difficulty:"Vanskelighet:", placeholder:"Skriv antall klosser",
        block:"klosser", disappear:"forsvinner", left:"hvor mange er igjen?", wrote:"Du skrev", correct:"(Riktig!)", wrong:"(Feil!)",
        difficulties:["Super Lett","Lett","Normal","Vanskelig","Umulig","Super Umulig"], footer:"Laget i Norge"},
    en:{title:"Block Game", difficulty:"Difficulty:", placeholder:"Type number of blocks",
        block:"blocks", disappear:"disappear", left:"how many are left?", wrote:"You wrote", correct:"(Correct!)", wrong:"(Wrong!)",
        difficulties:["Super Easy","Easy","Normal","Hard","Impossible","Super Impossible"], footer:"Made in Norway"},
    uk:{title:"Гра з блоками", difficulty:"Рівень складності:", placeholder:"Введіть кількість блоків",
        block:"блоків", disappear:"зникає", left:"скільки залишилось?", wrote:"Ви ввели", correct:"(Правильно!)", wrong:"(Неправильно!)",
        difficulties:["Супер Легко","Легко","Нормально","Важко","Неможливо","Супер Неможливо"], footer:"Зроблено в Норвегії"},
    ru:{title:"Игра с блоками", difficulty:"Сложность:", placeholder:"Введите количество блоков",
        block:"блоков", disappear:"исчезает", left:"сколько осталось?", wrote:"Вы ввели", correct:"(Правильно!)", wrong:"(Неправильно!)",
        difficulties:["Супер Легко","Легко","Нормально","Сложно","Невозможно","Супер Невозможно"], footer:"Сделано в Норвегии"},
    ar:{title:"لعبة المكعبات", difficulty:"الصعوبة:", placeholder:"اكتب عدد المكعبات",
        block:"مكعبات", disappear:"تختفي", left:"كم تبقى؟", wrote:"كتبت", correct:"(صحيح!)", wrong:"(خطأ!)",
        difficulties:["سهل جداً","سهل","عادي","صعب","مستحيل","مستحيل جداً"], footer:"مصنوع في النرويج"},
    sv:{title:"Blockspel", difficulty:"Svårighet:", placeholder:"Skriv antal block",
        block:"block", disappear:"försvinner", left:"hur många finns kvar?", wrote:"Du skrev", correct:"(Rätt!)", wrong:"(Fel!)",
        difficulties:["Super Lätt","Lätt","Normal","Svår","Omöjlig","Super Omöjlig"], footer:"Tillverkad i Norge"},
    pl:{title:"Gra w klocki", difficulty:"Trudność:", placeholder:"Wpisz liczbę klocków",
        block:"klocków", disappear:"znika", left:"ile zostało?", wrote:"Wpisałeś", correct:"(Poprawnie!)", wrong:"(Źle!)",
        difficulties:["Super Łatwo","Łatwo","Normalnie","Trudno","Niemożliwe","Super Niemożliwe"], footer:"Wyprodukowano w Norwegii"},
    da:{title:"Klodsspil", difficulty:"Sværhedsgrad:", placeholder:"Skriv antal klodser",
        block:"klodser", disappear:"forsvinder", left:"hvor mange er tilbage?", wrote:"Du skrev", correct:"(Korrekt!)", wrong:"(Forkert!)",
        difficulties:["Super Let","Let","Normal","Svær","Umulig","Super Umulig"], footer:"Fremstillet i Norge"},
    ur:{title:"بلاک کھیل", difficulty:"مشکل:", placeholder:"بلاکس کی تعداد لکھیں",
        block:"بلاکس", disappear:"غائب ہوجاتے ہیں", left:"کتنے باقی ہیں؟", wrote:"آپ نے لکھا", correct:"(صحیح!)", wrong:"(غلط!)",
        difficulties:["بہت آسان","آسان","عام","مشکل","ناممکن","بہت ناممکن"], footer:"ناروے میں بنایا گیا"}
};

const difficultyKeys = ["superEasy","easy","normal","trickyHard","impossible","superImpossible"];

function setLanguage(l){
    lang = l;
    document.getElementById("title").innerText = text[lang].title;
    document.getElementById("difficultyTitle").innerText = text[lang].difficulty;
    document.getElementById("answer").placeholder = text[lang].placeholder;
    document.getElementById("footer").innerText = text[lang].footer;
    renderDifficultyButtons();
    document.getElementById("languageSelect").value = lang;
    if(difficulty) newTask();
}

function renderDifficultyButtons(){
    const container = document.getElementById("difficultyButtons");
    container.innerHTML="";
    difficultyKeys.forEach((key,index)=>{
        let btn = document.createElement("button");
        btn.classList.add("btn");
        btn.innerText = text[lang].difficulties[index];
        btn.onclick = ()=>setDifficulty(key);
        container.appendChild(btn);
    });
}

function setDifficulty(d){
    difficulty = d;
    level = 1;
    xp = 0;
    totalXP = 0;
    updateXPBar();
    updateAchievements();
    document.getElementById("level").innerHTML = "Level: " + level;
    newTask();
}

function newTask(){
    let maxStart, maxRemove;
    switch(difficulty){
        case "superEasy": maxStart=3; maxRemove=1; break;
        case "easy": maxStart=6+level; maxRemove=4+level; break;
        case "normal": maxStart=12+level*2; maxRemove=10+level; break;
        case "trickyHard": maxStart=5; maxRemove=2; break;
        case "impossible": maxStart=200+level*20; maxRemove=180+level*15; break;
        case "superImpossible": maxStart=500+level*50; maxRemove=480+level*40; break;
        default: maxStart=6; maxRemove=3;
    }

    let start = Math.floor(Math.random()*maxStart)+2;
    let remove = Math.floor(Math.random()*Math.min(start-1,maxRemove))+1;
    currentAnswer = start-remove;

    showBlocks(start,remove);

    let label = text[lang].difficulties[difficultyKeys.indexOf(difficulty)];
    document.getElementById("task").innerHTML = `${start} ${text[lang].block}, ${remove} ${text[lang].disappear} – ${text[lang].left} (${label})`;

    document.getElementById("answer").value = "";
    document.getElementById("result").innerHTML = "";
}

function showBlocks(start, remove){
    const container = document.getElementById("blocks");
    container.innerHTML="";
    for(let i=0;i<start;i++){
        let b=document.createElement("div");
        b.classList.add("block");
        container.appendChild(b);
    }
}

function checkAnswer(){
    let a = parseInt(document.getElementById("answer").value);
    if(isNaN(a)) return;

    document.getElementById("result").innerHTML = `${text[lang].wrote}: ${a} ${text[lang].block} ` +
        (a===currentAnswer ? `<span style="color:#00ff8c">${text[lang].correct}</span>` 
                           : `<span style="color:#ff5050">${text[lang].wrong}</span>`);

    if(a===currentAnswer){
        xp += 5;
        totalXP += 5;
        while(xp>=xpToLevel){
            xp -= xpToLevel;
            level++;
            document.getElementById("level").innerHTML = "Level: " + level;
        }
        updateXPBar();
        updateAchievements();
        setTimeout(()=>newTask(),500);
    }
}

function updateXPBar(){
    let percent = (xp/xpToLevel)*100;
    document.getElementById("xp-bar").style.width = percent + "%";
}

function updateAchievements(){
    if(level>=5) document.getElementById("ach-level5").classList.add("unlocked");
    if(level>=10) document.getElementById("ach-level10").classList.add("unlocked");
    if(totalXP>=100) document.getElementById("ach-100xp").classList.add("unlocked");
}

// Initial render
setLanguage(lang);
</script>

</body>
</html>
