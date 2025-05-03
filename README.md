<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Интерактивная Канбан-Доска</title>
  <style>
    body { font-family: sans-serif; background: #f0f0f0; margin: 0; padding: 20px; }
    .board {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 10px;
    }
    .cell {
      background: #fff;
      border: 1px solid #ccc;
      padding: 10px;
      min-height: 120px;
      position: relative;
      overflow-y: auto;
    }
    .cell h4 {
      margin: 0 0 10px 0;
      font-size: 14px;
    }
    .card {
      background: #ffeeba;
      border-left: 5px solid red;
      padding: 8px;
      margin-bottom: 6px;
      cursor: move;
    }
    .card.green { border-left-color: green; background: #d4edda; }
    .card.red { border-left-color: red; background: #f8d7da; }
    .controls {
      margin-top: 20px;
    }
    input, select, button {
      margin-right: 5px;
      padding: 5px;
    }
    label {
      font-size: 12px;
    }
    .card .edit-button, .card .delete-button {
      font-size: 12px;
      background: transparent;
      border: none;
      cursor: pointer;
    }
    .card .delete-button {
      color: red;
    }
  </style>
</head>
<body>

<h2>Интерактивная Канбан-Доска Броней</h2>
<div class="controls">
  <input id="name" placeholder="Имя">
  <input id="phone" placeholder="Телефон">
  <select id="table">
    <option disabled selected>Стол</option>
    <option value="1">1</option>
    <option value="2">2</option>
    <option value="3">3</option>
    <option value="4">4</option>
    <option value="5">5</option>
    <option value="6">6</option>
    <option value="7">7</option>
    <option value="8">8</option>
  </select>
  <input id="time" type="time" placeholder="Время брони">
  <input id="guests" type="number" placeholder="Количество человек">
  <label><input id="hookah" type="checkbox"> Кальян</label>
  <label><input id="vr" type="checkbox"> VR</label>
  <button onclick="addCard()">Добавить бронь</button>
</div>

<div class="board" id="board">
  <div class="cell" id="table-1"><h4>Стол 1</h4></div>
  <div class="cell" id="table-2"><h4>Стол 2</h4></div>
  <div class="cell" id="table-3"><h4>Стол 3</h4></div>
  <div class="cell" id="table-4"><h4>Стол 4</h4></div>
  <div class="cell" id="table-5"><h4>Стол 5</h4></div>
  <div class="cell" id="table-6"><h4>Стол 6</h4></div>
  <div class="cell" id="table-7"><h4>Стол 7</h4></div>
  <div class="cell" id="table-8"><h4>Стол 8</h4></div>
</div>

<script>
// Считывание данных из LocalStorage при загрузке страницы
window.onload = function() {
  console.log("Page loaded, checking LocalStorage data...");
  setTimeout(() => {
    loadDataFromLocalStorage();
  }, 1000); // Добавляем задержку 1 секунда
};

let cardId = 0;

function allowDrop(ev) {
  ev.preventDefault();
}

function drag(ev) {
  ev.dataTransfer.setData("text", ev.target.id);
}

function drop(ev) {
  ev.preventDefault();
  var data = ev.dataTransfer.getData("text");
  ev.target.closest('.cell').appendChild(document.getElementById(data));
}

function addCard() {
  const name = document.getElementById('name').value;
  const phone = document.getElementById('phone').value;
  const table = document.getElementById('table').value;
  const time = document.getElementById('time').value;
  const guests = document.getElementById('guests').value;
  const hookah = document.getElementById('hookah').checked ? 'Кальян' : '';
  const vr = document.getElementById('vr').checked ? 'VR' : '';
  
  if (!name || !phone || !table || !time || !guests) {
    alert("Заполните все поля!");
    return;
  }

  // Сохраняем карточку в LocalStorage
  const cardData = { name, phone, time, table, guests, hookah, vr, status: 'Ожидается' };
  saveDataToLocalStorage(cardData);

  // Создание карточки
  const card = document.createElement('div');
  card.className = 'card red';
  card.id = 'card' + cardId++;
  card.draggable = true;
  card.ondragstart = drag;

  card.innerHTML = `
    <strong>${name}</strong><br>
    Телефон: ${phone}<br>
    Время: ${time}<br>
    Количество: ${guests}<br>
    Стол: ${table}<br>
    ${hookah ? `<span>🍹 ${hookah}</span><br>` : ''}
    ${vr ? `<span>🎮 ${vr}</span><br>` : ''}
    <button class="edit-button" onclick="editCard(this.parentNode)">Редактировать</button>
    <button class="delete-button" onclick="deleteCard(this.parentNode)">Удалить</button>
    <button onclick="toggleStatus(this.parentNode)">Статус</button>
  `;

  // Используем правильный ID для столов
  const tableCell = document.querySelector(`#table-${table}`);
  if (tableCell) {
    tableCell.appendChild(card);
    console.log(`Карточка добавлена в Стол ${table}`); // Лог для отладки
  } else {
    console.log(`Ошибка: Стол ${table} не найден!`);  // Лог для отладки
  }
}

// Сохранение данных в LocalStorage
function saveDataToLocalStorage(cardData) {
  let existingData = JSON.parse(localStorage.getItem('bookings')) || [];
  existingData.push(cardData);
  localStorage.setItem('bookings', JSON.stringify(existingData));
  console.log('Data saved to LocalStorage:', cardData);  // Лог для отладки
}

// Загрузка данных из LocalStorage
function loadDataFromLocalStorage() {
  const storedData = JSON.parse(localStorage.getItem('bookings')) || [];
  console.log('Loaded data from LocalStorage:', storedData);  // Лог для отладки
  storedData.forEach(data => {
    const card = document.createElement('div');
    card.className = 'card red';
    card.id = 'card' + cardId++;
    card.draggable = true;
    card.ondragstart = drag;

    card.innerHTML = `
      <strong>${data.name}</strong><br>
      Телефон: ${data.phone}<br>
      Время: ${data.time}<br>
      Количество: ${data.guests}<br>
      Стол: ${data.table}<br>
      ${data.hookah ? `<span>🍹 ${data.hookah}</span><br>` : ''}
      ${data.vr ? `<span>🎮 ${data.vr}</span><br>` : ''}
      <button class="edit-button" onclick="editCard(this.parentNode)">Редактировать</button>
      <button class="delete-button" onclick="deleteCard(this.parentNode)">Удалить</button>
      <button onclick="toggleStatus(this.parentNode)">Статус</button>
    `;

    // Используем правильный ID для столов
    const tableCell = document.querySelector(`#table-${data.table}`);
    if (tableCell) {
      tableCell.appendChild(card);
    } else {
      console.log(`Ошибка: Стол ${data.table} не найден!`);  // Лог для отладки
    }
  });
}

function toggleStatus(card) {
  if (card.classList.contains('red')) {
    card.classList.remove('red');
    card.classList.add('green');
  } else {
    card.classList.remove('green');
    card.classList.add('red');
  }
}

function deleteCard(card) {
  let existingData = JSON.parse(localStorage.getItem('bookings')) || [];
  existingData = existingData.filter(data => data.name !== card.querySelector('strong').textContent);
  localStorage.setItem('bookings', JSON.stringify(existingData));
  console.log('Data removed from LocalStorage:', card.querySelector('strong').textContent);  // Лог для отладки
  card.remove();
}

function editCard(card) {
  const name = prompt("Имя", card.querySelector("strong").textContent);
  const phone = prompt("Телефон", card.querySelector("br").nextSibling.textContent.split(":")[1]);
  const time = prompt("Время", card.querySelector("br").nextSibling.nextSibling.textContent.split(":")[1]);
  const guests = prompt("Количество человек", card.querySelector("br").nextSibling.nextSibling.nextSibling.textContent.split(":")[1]);
  
  card.querySelector("strong").textContent = name;
  card.querySelector("br").nextSibling.textContent = "Телефон: " + phone;
  card.querySelector("br").nextSibling.nextSibling.textContent = "Время: " + time;
  card.querySelector("br").nextSibling.nextSibling.nextSibling.textContent = "Количество: " + guests;

  // Сохранение изменений в LocalStorage
  let existingData = JSON.parse(localStorage.getItem('bookings')) || [];
  existingData = existingData.map(data => {
    if (data.name === card.querySelector("strong").textContent) {
      data.name = name;
      data.phone = phone;
      data.time = time;
      data.guests = guests;
    }
    return data;
  });
  localStorage.setItem('bookings', JSON.stringify(existingData));
  console.log('Data updated in LocalStorage:', existingData);  // Лог для отладки
}
</script>

</body>
</html>
