const Tags = [
    { id: 'edging', title: 'Edging/Denial' },
    { id: 'cum', title: 'Orgasm' },
    { id: 'pain_light', title: 'Pain (light)' },
    { id: 'pain_extreme', title: 'Pain (extreme)' },
    { id: 'piss', title: 'Piss' },
    { id: 'choking', title: 'Breathplay' },
    { id: 'public_discrete', title: 'Public (discrete)' },
    { id: 'public_risky', title: 'Public (risky)' },
    { id: 'bondage', title: 'Bondage' },
    { id: 'bodywriting', title: 'Body writing' },
    { id: 'speech', title: 'Speech control' },
    { id: 'clothes', title: 'Clothing control' },
    { id: 'longterm', title: 'Long-term' },
    { id: 'chores', title: 'Chores' },
    { id: 'petplay', title: 'Petplay' },
    { id: 'bimbo', title: 'Bimbofication' },
    { id: 'feminization', title: 'Feminization' },
    { id: 'hypno', title: 'Hypnosis/Mind control' },
    { id: 'temperature', title: 'Temperature play' },
    { id: 'food', title: 'Food' },
];

const BodyParts = [
    { id: 'pussy', title: 'Pussy' },
    { id: 'clit', title: 'Clit' },
    { id: 'tits', title: 'Tits' },
    { id: 'nipples', title: 'Nipples' },
    { id: 'ass', title: 'Ass' },
    { id: 'cock', title: 'Cock' },
    { id: 'balls', title: 'Balls' },
    { id: 'mouth', title: 'Mouth' },
];

const Toys = [
    { id: 'dildo', title: 'Dildo' },
    { id: 'gag', title: 'Gag' },
    { id: 'dildo_gag', title: 'Dildo gag' },
    { id: 'buttplug', title: 'Buttplug' },
    { id: 'ropes', title: 'Ropes or tape' },
    { id: 'clamps', title: 'Clamps or clothespins' },
    { id: 'cockcage', title: 'Cock cage' },
    { id: 'fleshlight', title: 'Fleshlight' },
    { id: 'vibe', title: 'Vibrator' },
    { id: 'bluetooth_toy', title: 'Bluetooth toy' },
];

var score = 0, finished = 0, failed = 0, skipped = 0;
var num_score, num_finished, num_failed, num_skipped;
var taskText;
var taskData;
var timer_text_div, timer_text, timer_button, finishButton;

var allowSkipping = true;
var enableTimer = true;

var currentTask;
var currentTasks = null;
var currentTasksIndex = 0;

var tags_array, bodyparts_array, toys_array;

var blockedTasksIds = [];

document.addEventListener("DOMContentLoaded", function () {
    timer_text_div = document.getElementById("i4i4h");
    timer_text = document.getElementById("ijuit");
    timer_button = document.getElementById("idc0l");
    
    finishButton = document.getElementById("iar3s");

    taskText = document.getElementById("taskText");
    
    num_score = document.getElementById("num_score");
    num_finished = document.getElementById("num_finished");
    num_failed = document.getElementById("num_failed");
    num_skipped = document.getElementById("num_skipped");
    loadScore();

    timer_text_div.style.display = "none";
    timer_button.style.display = "none";

    const tags_list = document.getElementById("tags_list");
    const bodyparts_list = document.getElementById("bodyparts_list");
    const toys_list = document.getElementById("toys_list");

    for (const el of Tags) {
        tags_list.innerHTML += '<input class="tags_checkboxes" type="checkbox" name="' + el.id + '" id="' + el.id + '"><label for="' + el.id +'">' + el.title + '</title><br>';
    }
    for (const el of BodyParts) {
        bodyparts_list.innerHTML += '<input class="bodyparts_checkboxes" type="checkbox" name="' + el.id + '" id="' + el.id + '"><label for="' + el.id +'">' + el.title + '</title><br>';
    }
    for (const el of Toys) {
        toys_list.innerHTML += '<input class="toys_checkboxes" type="checkbox" name="' + el.id + '" id="' + el.id + '"><label for="' + el.id +'">' + el.title + '</title><br>';
    }

    loadCheckboxState("tags_checkboxes");
    loadCheckboxState("bodyparts_checkboxes");
    loadCheckboxState("toys_checkboxes");

    allowSkipping = localStorage.getItem("allowSkipping") == "true" || localStorage.getItem("allowSkipping") == null;
    document.getElementById("i4mla").style.display = allowSkipping ? "inline-block" : "none";
    document.getElementById("dont_allow_skipping").checked = !allowSkipping;

    enableTimer = localStorage.getItem("enableTimer") == "true" || localStorage.getItem("enableTimer") == null;
    document.getElementById("disable_timer").checked = !enableTimer;

    parseCSV(false);

    loadBlockedIds();
});

function parseCSV(showAlert = true) {
    const fileInput = document.getElementById('csvFileInput');
    const file = fileInput.files[0];

    if (!file) {
        if (showAlert) alert("Upload a .csv spreadsheet with tasks first.");
        return;
    }
    else
    {
        const reader = new FileReader();
    
        reader.onload = function (event) {
            const csvData = event.target.result;
            taskData = csvToObj(csvData);
        };
    
        reader.readAsText(file);
    }  
}

function csvToObj(csv) {
    const lines = csv.split('\n');
    const headers = parseLine(lines[0]);
    const data = [];

    for (let i = 1; i < lines.length; i++) {
        const line = lines[i].trim();
        if (line) {
            const values = parseLine(line);
            const obj = {};
            headers.forEach((header, index) => {
                obj[header] = values[index] || '';
            });
            data.push(obj);

            obj.Tags = obj.Tags.toLowerCase().split(",").map(item=>item.trim());
            obj.BodyParts = obj.BodyParts.toLowerCase().split(",").map(item=>item.trim());
            obj.Toys = obj.Toys.toLowerCase().split(",").map(item => item.trim());
            if (obj.Tags.length == 1 && obj.Tags[0] == "") obj.Tags = [];
            if (obj.BodyParts.length == 1 && obj.BodyParts[0] == "") obj.BodyParts = [];
            if (obj.Toys.length == 1 && obj.Toys[0] == "") obj.Toys = [];
        }
    }
    return data;
}

function parseLine(strData) {

    const objPattern = new RegExp(("(\\,|\\r?\\n|\\r|^)(?:\"([^\"]*(?:\"\"[^\"]*)*)\"|([^\\,\\r\\n]*))"),"gi");
    let arrMatches = null, arrData = [[]];
    while (arrMatches = objPattern.exec(strData)){
        if (arrMatches[1].length && arrMatches[1] !== ",")arrData.push([]);
        arrData[arrData.length - 1].push(arrMatches[2] ? 
            arrMatches[2].replace(new RegExp( "\"\"", "g" ), "\"") :
            arrMatches[3]);
    }
    return arrData[0];
}

function startTaskTimer() {   
    startTimer(60 * currentTask.Minutes, timer_text);
}

function finishTask() {
    finished++;
    score++;
    updateScore();

    if (currentTask.FinishedMessage != null && currentTask.FinishedMessage != "") {
        document.getElementById('i11qb').style.display = 'none';
        
        clearInterval(timer_interval);
        timer_text_div.style.display = "none";
        timer_button.style.display = "none";

        taskText.innerHTML = "<i>Because you finished the task: </i><br>" + currentTask.FinishedMessage + '<br><a class="gjs-button" id="i4mla2" onclick="nextTask();">Next task</a>';
    } else {
        nextTask();
    }
}

function failTask() {
    failed++;
    score--;
    updateScore();

    if (currentTask.FailedMessage != null && currentTask.FailedMessage != "") {
        document.getElementById('i11qb').style.display = 'none';
        
        clearInterval(timer_interval);
        timer_text_div.style.display = "none";
        timer_button.style.display = "none";

        taskText.innerHTML = "<i>Because you failed the task: </i><br>" + currentTask.FailedMessage + '<br><a class="gjs-button" id="i4mla2" onclick="nextTask();">Next task</a>';
    } else {
        nextTask();
    }
}

function skipTask() {
    skipped++;
    updateScore();
    nextTask();
}

function resetScore() {
    score = 0;
    skipped = 0;
    failed = 0;
    finished = 0;
    updateScore();
}

function blockTask() {
    blockedTasksIds.push(currentTask.ID);
    localStorage.setItem("blockedTasksIds", JSON.stringify(blockedTasksIds));
    getCurrentTasks();
    nextTask();
}
function resetBlocked() {
    blockedTasksIds = [];
    localStorage.setItem("blockedTasksIds", JSON.stringify(blockedTasksIds));
    getCurrentTasks();
}

function updateAllowSkip() {
    allowSkipping = !document.getElementById("dont_allow_skipping").checked;
    localStorage.setItem('allowSkipping', allowSkipping);
    document.getElementById("i4mla").style.display = allowSkipping ? "inline-block" : "none";
}

function updateEnableTimer() {
    enableTimer = !document.getElementById("disable_timer").checked;
    localStorage.setItem('enableTimer', enableTimer);
}

function nextTask() {
    clearInterval(timer_interval);
    timer_text_div.style.display = "none";
    timer_button.style.display = "none";
    
    if (currentTasks == null) getCurrentTasks();
    
    var task = currentTasks[currentTasksIndex];
    currentTasksIndex++;
    if (currentTasksIndex+1 > currentTasks.length) currentTasksIndex = 0;
        
    if (task) {
        document.getElementById('i11qb').style.display = 'flex';

        console.log(task);

        timer_text.innerHTML = task.Minutes.toString().padStart(2, 0) + ":00";
        taskText.innerHTML = task.TaskText;
        currentTask = task;
        if (enableTimer && task.Minutes != "" && task.Minutes > 0) {
            timer_text_div.style.display = "initial";
            timer_button.style.display = "inline-block";

            if (currentTask.Type == "TimeLimit") {
                taskText.innerHTML += "<br><br><i>This task has a time limit. If you don't click Finish before the timer runs out, the task will automatically fail.</i>";
            }
            else if (currentTask.Type == "Timer") {
                finishButton.style.display = "none";
                taskText.innerHTML += "<br><br><i>This task has a timer. The Finish button will only appear after the timer runs out.</i>";
            }
        }
    }
    else {
        alert("There are no tasks that match your settings. Try selecting more tags toys and bodyparts, add more tasks to the spreadsheet, or reset blocked tasks.");
    }
}

function getCurrentTasks() {
    if (taskData == null)
    {
        alert("Upload a .csv spreadsheet with tasks first.");
        return;
    }

    currentTasksIndex = 0;
    currentTasks = taskData.filter(function (el) {
        return !blockedTasksIds.includes(el.ID) && el.Tags.every(item => tags_array.includes(item)) && el.BodyParts.every(item => bodyparts_array.includes(item)) && el.Toys.every(item => toys_array.includes(item));
    });
    shuffle(currentTasks);
}

function shuffle(array) {
    let currentIndex = array.length;
    while (currentIndex != 0) {
      let randomIndex = Math.floor(Math.random() * currentIndex);
      currentIndex--;  
      [array[currentIndex], array[randomIndex]] = [array[randomIndex], array[currentIndex]];
    }
}

function updateScore() {
    num_score.innerHTML = score;
    num_finished.innerHTML = finished;
    num_failed.innerHTML = failed;
    num_skipped.innerHTML = skipped;
    saveScore();
}

function saveCheckboxState(classname) {
    const checkboxes = document.querySelectorAll('.' + classname);
    const checkboxStates = {};

    checkboxes.forEach(checkbox => {
      checkboxStates[checkbox.id] = checkbox.checked;
    });

    localStorage.setItem(classname, JSON.stringify(checkboxStates));

    setVariables(classname);
}

function setVariables(classname) {
    var checked = document.querySelectorAll('.' + classname + ':checked');
    checked = Array.from(checked, node => node.id);
    switch (classname) {
        case "tags_checkboxes":
            tags_array = checked;
            break;
        case "bodyparts_checkboxes":
            bodyparts_array = checked;
            break;
        case "toys_checkboxes":
            toys_array = checked;
            break;
    }
}

function loadBlockedIds() {
    const json = localStorage.getItem("blockedTasksIds");
    if (json) {
        blockedTasksIds = JSON.parse(json);
    }
}

function loadCheckboxState(classname) {
    const savedState = localStorage.getItem(classname);

    if (savedState) {
      const checkboxStates = JSON.parse(savedState);
      const checkboxes = document.querySelectorAll('.' + classname);
        
      checkboxes.forEach(checkbox => {
        if (checkboxStates[checkbox.id] !== undefined) {
          checkbox.checked = checkboxStates[checkbox.id];
        }
      });
    }

    setVariables(classname);
}

function saveScore() {
    const scores = {
        score: score,
        finished: finished,
        failed: failed,
        skipped: skipped
    };
    localStorage.setItem('scores', JSON.stringify(scores));
}

function loadScore() {
    const savedVariables = localStorage.getItem('scores');
    
    if (savedVariables) {
      const { score: loadedA, finished: loadedB, failed: loadedC, skipped: loadedD } = JSON.parse(savedVariables);
      
      score = loadedA !== undefined ? loadedA : score;
      finished = loadedB !== undefined ? loadedB : finished;
      failed = loadedC !== undefined ? loadedC : failed;
      skipped = loadedD !== undefined ? loadedD : skipped;
    }
    updateScore();
}

function resetFilters()
{
    if (confirm("Do you want to reset all filters?")) {
        var all_tags = document.querySelectorAll('.tags_checkboxes, .bodyparts_checkboxes, .toys_checkboxes'); for (var checkbox in all_tags) all_tags[checkbox].checked = false;
    }
}

function buyReward(price)
{
    if (score >= price) {
        score -= 5;
        updateScore();
        alert("Reward bought!");
    }
    else {
        alert("You don't have enough points.");
    }
}

var timer_interval;
function startTimer(duration, display) {
    timer_button.style.display = "none";
    clearInterval(timer_interval);

    var timer = duration, minutes, seconds;
    timer_interval = setInterval(function () {
        minutes = parseInt(timer / 60, 10);
        seconds = parseInt(timer % 60, 10);

        minutes = minutes < 10 ? "0" + minutes : minutes;
        seconds = seconds < 10 ? "0" + seconds : seconds;

        display.textContent = minutes + ":" + seconds;

        if (--timer < 0) {
            timer = duration;
        }

        if (minutes == 0 && seconds == 0) {
            beep();
            if (currentTask.Type == "TimeLimit") {
                failTask();
            }
            else if (currentTask.Type == "Timer") {
                finishButton.style.display = "inline-block";
            }
        }
    }, 1000);
}

function beep() {
    var audio = new Audio('timer.ogg');
    audio.play();
}
