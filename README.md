<style>

img {
    width:22px;
    float:left; 
     margin:0;
}
label { display: block; width: 400px; }
</style>          

          
<div id="options" class="login">
<div>
    <label>Пользователи (через запятую)<br>
    <textarea id="voteaccounts" required rows="15" cols="40" placeholder="Аккаунты за которые голосовать. Через запятую" onkeyup="showAccounts(this)">ropox</textarea></label>
    <br>
    <label>Логин пользователя<br>
    <input id="username" type="text" required class="text" name="username" placeholder="Логин пользователя"></label>
    <br>
    <label for="k">Постинг ключ</label>
    <input id="k" type="password" required class="text" name="password" placeholder="Постинг ключ, должен начинаться с '5', а не с 'GLS'" size="50"><button onclick="toggleKey()" title="Показать ключ">*</button><br>
    <br>
    <label>Сила голоса %<br>
    <input id="votepower" type="number" class="text" value="100" name="text" min="0" max="100" size="15" placeholder="Сила голоса %"></label>
    <br>
    <label>Голосовать за пост по прошествии N минут<br>
    <input id="delay" type="number" class="text" value="5" name="text" min="0" size="15" placeholder="Задержка мин."></label>
    <input id="debug" type="hidden" value="off"/>
</div>
<br>
<button onclick="startVoting()" class="signin">
Запуск голосования
</button>
<button onclick="stopVoting()" class="signin">
Остaновить голосование
</button>
<div id="progress"></div>
<hr>
</div>
<div id="accounts_view"></div>
<div id="nicedata"></div>


<script src="golos.js"></script>
<script src="votebot.js" onload="recoverData()"></script>
