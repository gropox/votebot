<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="utf-8"/>
<title>Vote Bot</title>
<meta name="Description" content="VoteBot для голоса">
<meta name="viewport" content="width=device-width,minimum-scale=1,initial-scale=1">
<link rel="icon" type="image/x-icon" href="https://golos.io/images/favicons/favicon.ico"/>
<script src="steem.min.js"></script>
</head>
<body onload="recoverData()">
<div id="options" class="login">
<h2 class="active"> VoteBot </h2>
<form>
    <textarea id="voteaccounts" required rows="15" cols="40" placeholder="Аккаунты за которые голосовать. Через запятую" onkeyup="showAccounts(this)">ropox</textarea>
    <br>
    <input id="username" type="text" required class="text" name="username" placeholder="Логин пользователя">
    <br>
    <input id="k" type="password" required class="text" name="password" placeholder="Постинг ключ" size="50">
    <br>
    <input id="votepower" type="number" value="100" class="text" name="text" min="0" max="100" size="15" placeholder="Сила голоса %">
</form>
<br>
<button onclick="startVoting()" class="signin">
Запуск голосования
</button>
<button onclick="stopVoting()" class="signin">
Остaновить голосование
</button>

<hr>
<div id="accounts_view"></div>

</div>
<div id="nicedata"></div>
<script type="text/javascript">
    
    var golosbase = "https://golos.io";
    var golos_ws = "wss://ws.golos.io";
    var voteQueue = [];
    
    function recoverData() {
        let raw_users = document.getElementById("voteaccounts");
        let key = document.getElementById("k");
        let username = document.getElementById("username");
        
        raw_users.value = localStorage.getItem("raw_users");
        key.value = localStorage.getItem("key");
        username.value = localStorage.getItem("username");
    }
    
    function parseAccounts(inp) {
        let accs = inp.split(",");
        for(let i = 0; i < accs.length; i++) {
            accs[i] = accs[i].trim();
        }
        return accs;
    }

    function showAccounts(ta) {
        var accs = parseAccounts(ta.value);
        var view = document.getElementById("accounts_view");
        var html = "";
        for(var i = 0; i < accs.length; i++) {
            html = html + "<br><a href='" + golosbase +"/@" + accs[i] + "'>@"+accs[i]+ "</a>";
        }
        view.innerHTML = html;
    }
    
    var votepower = 0;
    var workerTimer;
    
    function startVoting() {
        console.log("startBot");
        steem.api.setWebSocket(golos_ws);
        var users = parseAccounts(document.getElementById("voteaccounts").value),
            k = document.getElementById("k").value,
            username = document.getElementById("username").value,
            votepower = document.getElementById("votepower").value,
            time, starttime, t = 1000,
            period = 10 * 60,
            utime, start, history,
            raw_users = document.getElementById("voteaccounts").value;

        localStorage.setItem("raw_users", raw_users);
        localStorage.setItem("key", k);
        localStorage.setItem("username", username);

        if(typeof users == "undefined" ||users.length == 0 || users.length == 1 && users[0] == "") {
            alert("Введите имена пользователейза которомы следить!");
            return;
        }

        //Инициализация кэша
        var accounts = {};
        for(let i = 0; i < users.length; i++) {
            accounts[users[i]] = {lastId:-1, queue: []}; //-1 неизвестно, ждем новых постов
        }
        console.log(accounts);

        var checkDelay = 10000;
        var votingDelay = 3000;
        workerTimer = setInterval(function() {

            //steem.api.getDynamicGlobalProperties(function(err, result) {
            //    starttime = Date.parse(result.time) / t;
            //});
            
            for(let i = 0; i < users.length; i++) {
                let u = users[i];
                console.log("get account history for " +u);
                steem.api.getAccountHistory(u, -1, 150, function(err, result) {
                    //получили 10 последних записей из истории 
                    //console.log(result);
                    for(var ai = 0; ai < result.length; ai++) {
                        let heId = result[ai][0];
                        //console.log(heId);
                        if(accounts[u].lastId < heId) {
                            let he = result[ai][1].op;
                            if(typeof he !== "undefined") {
                                let op = he[0];
                                let entry = he[1];
                                //console.log(op);
                                if(op == "comment" && entry.author == u && entry.parent_author == "" && !entry.body.match("^@@ .* @@")) {
                                    //console.log(entry);
                                    console.log(heId + ":" + accounts[u].lastId  + " add vote to queue: " + u + " / " + entry.permlink);
                                    accounts[u].queue.push({
                                        author : u,
                                        permlink : entry.permlink,
                                        title : entry.title
                                    });
                                    checkDelay = checkDelay + votingDelay;
                                }
                            }
                            accounts[u].lastId = heId;
                        }
                    }
                    //недавние голосования
                    var actualVotes = [];
                    var goVote = setInterval(function() {
                        let vote = accounts[u].queue.shift();
                        
                        if(typeof vote !== "undefined") {

                            //На всякий случай исключить двойное голосование
                            let voteKey = vote.author + "/" + vote.permlink;
                            if(!actualVotes.includes(voteKey)) {
                                actualVotes.push(voteKey);
                                
                                //убедиться, может быть раньше голосовали
                                steem.api.getActiveVotes(vote.author, vote.permlink, function(err, result) {
                                         
                                    //console.log(result);
                                    var alreadyVoted = false;
                                    for(let i = 0; i < result.length; i++) {
                                        if(result[i].voter == username) {
                                            alreadyVoted = true;
                                            break;
                                        }
                                    }
                                    if(!alreadyVoted) {
                                        votehtml = '<div id="item" class="myJson"><a href="https://golos.io/@' + vote.author + '/' + vote.permlink + '"><strong>' + vote.author + ": "  + vote.title + '</a></div>';
                                        document.getElementById('nicedata').insertAdjacentHTML('afterbegin', votehtml);
                                        steem.broadcast.vote(k, username, vote.author, vote.permlink, votepower * 100, function(err, result) {
                                            console.log(err,result);
                                        });
                                        if(checkDelay > 10000) {
                                            checkDelay = checkDelay - votingDelay;
                                        }
                                    }
                                });
                            }
                        } else {
                            clearInterval(goVote);
                        }

                    }, votingDelay);
                });
            }
        }, checkDelay);
    }
        
    function stopVoting() {
        clearInterval(workerTimer);
    }
        
</script>
</body>
</html>
