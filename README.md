<style>

img {
    width:22px;
    float:left; 
     margin:0;
}

</style>          

          
<div id="options" class="login">
<div>
    <textarea id="voteaccounts" required rows="15" cols="40" placeholder="Аккаунты за которые голосовать. Через запятую" onkeyup="showAccounts(this)">ropox</textarea>
    <br>
    <input id="username" type="text" required class="text" name="username" placeholder="Логин пользователя">
    <br>
    <input id="k" type="password" required class="text" name="password" placeholder="Постинг ключ" size="50"><button onclick="toggleKey()" title="Показать ключ">*</button>
    <br>
    <input id="votepower" type="number" value="100" class="text" name="text" min="0" max="100" size="15" placeholder="Сила голоса %">
</div>
<br>
<button onclick="startVoting()" class="signin">
Запуск голосования
</button>
<button onclick="stopVoting()" class="signin">
Остaновить голосование
</button>

<hr>
</div>
<div id="accounts_view"></div>
<div id="nicedata"></div>


<script src="golos.js"></script>
<script src="votebot.js" onload="recoverData()"></script>
