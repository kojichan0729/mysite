<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Todo管理アプリ（保存＆期限付き）</title>
  <style>
    body {
      font-family: sans-serif;
      text-align: center;
      padding: 20px;
      max-width: 600px;
      margin: auto;
    }

    input, button, select, textarea {
      padding: 10px;
      font-size: 16px;
      margin: 5px;
      width: 90%;
      max-width: 300px;
      box-sizing: border-box;
    }

    ul {
      list-style: none;
      padding: 0;
      margin-top: 20px;
    }

    li {
      padding: 10px;
      margin: 5px 0;
      border-bottom: 1px solid #ccc;
      border-radius: 8px;
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      text-align: left;
    }

    .completed {
      text-decoration: line-through;
      color: gray;
      opacity: 0.6;
    }

    .deadline {
      font-size: 14px;
      color: #666;
      margin-top: 5px;
    }

    .deleteBtn {
      background: red;
      color: white;
      border: none;
      border-radius: 5px;
      padding: 5px 10px;
      cursor: pointer;
      margin-top: 5px;
    }

    .deleteBtn:hover {
      background: darkred;
    }

    .category-仕事 {
      background-color: #ffe0e0;
    }

    .category-勉強 {
      background-color: #e0f0ff;
    }

    .category-遊び {
      background-color: #e0ffe0;
    }

    .filterBtns, .sortBtns {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      margin-top: 10px;
    }

    button {
      min-width: 100px;
      margin: 5px;
    }

    @media screen and (max-width: 480px) {
      li {
        font-size: 14px;
      }

      input, select, button, textarea {
        font-size: 14px;
      }

      .deadline, .deleteBtn {
        margin-top: 5px;
      }
    }
  </style>
</head>
<body>

  <h1>Todo管理アプリ</h1>

  <input type="text" id="todoInput" placeholder="やることを入力" />
  <input type="date" id="deadlineInput" />
  <select id="categoryInput">
    <option value="仕事">仕事</option>
    <option value="勉強">勉強</option>
    <option value="遊び">遊び</option>
  </select>
  <textarea id="memoInput" placeholder="メモを入力" rows="3"></textarea>
  <select id="priorityInput">
    <option value="1">★☆☆</option>
    <option value="2">★★☆</option>
    <option value="3">★★★</option>
  </select>
  <button id="addBtn">追加</button>

  <div class="filterBtns">
    <button class="filterBtn" data-category="all">すべて</button>
    <button class="filterBtn" data-category="仕事">仕事</button>
    <button class="filterBtn" data-category="勉強">勉強</button>
    <button class="filterBtn" data-category="遊び">遊び</button>
  </div>

  <div class="sortBtns">
    <button id="sortByPriority">重要度順</button>
    <button id="sortByDeadline">期限順</button>
  </div>

  <ul id="todoList"></ul>

  <script>
    const todoInput = document.getElementById("todoInput");
    const deadlineInput = document.getElementById("deadlineInput");
    const categoryInput = document.getElementById("categoryInput");
    const memoInput = document.getElementById("memoInput");
    const priorityInput = document.getElementById("priorityInput");
    const addBtn = document.getElementById("addBtn");
    const todoList = document.getElementById("todoList");
    const sortByPriority = document.getElementById("sortByPriority");
    const sortByDeadline = document.getElementById("sortByDeadline");

    let todos = JSON.parse(localStorage.getItem("todos") || "[]");
    let currentFilter = "all";

    function saveTodos() {
      localStorage.setItem("todos", JSON.stringify(todos));
    }

    function getStars(priority) {
      return "★".repeat(priority) + "☆".repeat(3 - priority);
    }

    function renderTodos() {
      todos.sort((a, b) => {
        if (a.done === b.done) return 0;
        return a.done ? 1 : -1;
      });

      todoList.innerHTML = "";

      todos
        .filter(todo => currentFilter === "all" || todo.category === currentFilter)
        .forEach((todo, index) => {
          const li = document.createElement("li");
          li.classList.add(`category-${todo.category}`);
          if (todo.done) li.classList.add("completed");

          const checkbox = document.createElement("input");
          checkbox.type = "checkbox";
          checkbox.checked = todo.done;
          checkbox.addEventListener("change", () => {
            todo.done = checkbox.checked;
            saveTodos();
            renderTodos();
          });

          const title = document.createElement("span");
          title.textContent = todo.text;

          const stars = document.createElement("span");
          stars.textContent = getStars(todo.priority || 1);
          stars.style.marginTop = "5px";

          const deadline = document.createElement("div");
          deadline.className = "deadline";
          deadline.textContent = `期限: ${todo.deadline || "なし"}`;

          const memo = document.createElement("div");
          memo.style.fontSize = "14px";
          memo.style.marginTop = "5px";
          memo.textContent = todo.memo || "";

          const deleteBtn = document.createElement("button");
          deleteBtn.className = "deleteBtn";
          deleteBtn.textContent = "削除";
          deleteBtn.addEventListener("click", () => {
            todos.splice(index, 1);
            saveTodos();
            renderTodos();
          });

          li.appendChild(checkbox);
          li.appendChild(title);
          li.appendChild(stars);
          li.appendChild(deadline);
          li.appendChild(memo);
          li.appendChild(deleteBtn);
          todoList.appendChild(li);
        });
    }

    document.querySelectorAll(".filterBtn").forEach(button => {
      button.addEventListener("click", () => {
        currentFilter = button.dataset.category;
        renderTodos();
      });
    });

    sortByPriority.addEventListener("click", () => {
      todos.sort((a, b) => b.priority - a.priority);
      renderTodos();
    });

    sortByDeadline.addEventListener("click", () => {
      todos.sort((a, b) => {
        if (!a.deadline) return 1;
        if (!b.deadline) return -1;
        return new Date(a.deadline) - new Date(b.deadline);
      });
      renderTodos();
    });

    addBtn.addEventListener("click", () => {
      const text = todoInput.value.trim();
      const deadline = deadlineInput.value;
      const category = categoryInput.value;
      const memo = memoInput.value.trim();
      const priority = parseInt(priorityInput.value);

      if (text === "") return;

      todos.push({ text, deadline, category, memo, priority, done: false });

      saveTodos();
      renderTodos();

      todoInput.value = "";
      deadlineInput.value = "";
      memoInput.value = "";
      priorityInput.value = "1";
    });

    renderTodos();
  </script>

</body>
</html>
