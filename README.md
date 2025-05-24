<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Expense Tracker (₹ INR) - Big & Small Amounts</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f0f0f0;
      padding: 40px;
    }
    .container {
      background: white;
      max-width: 750px;
      margin: auto;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    h2 {
      text-align: center;
      margin-bottom: 20px;
    }
    input, select, button {
      padding: 10px;
      margin: 5px 0;
      width: 100%;
      font-size: 1em;
    }
    .expense-section {
      margin-top: 30px;
    }
    .expense-section h3 {
      border-bottom: 2px solid #007bff;
      padding-bottom: 5px;
    }
    .expense-item {
      background: #e9ecef;
      margin: 10px 0;
      padding: 10px;
      border-radius: 5px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
    }
    .actions button {
      margin-left: 5px;
    }
    .total {
      font-weight: bold;
      margin-top: 10px;
      text-align: right;
      color: #333;
    }
    .search-filter {
      display: flex;
      gap: 10px;
      margin: 10px 0;
      flex-wrap: wrap;
    }
    .tag {
      background: #007bff;
      color: white;
      border-radius: 3px;
      padding: 2px 6px;
      font-size: 0.85em;
      margin-left: 5px;
      user-select: none;
    }
  </style>
</head>
<body>

<div class="container">
  <h2>Expense Tracker (₹ INR) - Big & Small Amounts</h2>

  <input type="text" id="expenseName" placeholder="Expense Name (e.g. Rent)">
  <input type="number" id="expenseAmount" placeholder="Amount in ₹ (e.g. 500)">
  <select id="expenseCategory">
    <option value="">Select Category</option>
    <option value="Food">Food</option>
    <option value="Utilities">Utilities</option>
    <option value="Travel">Travel</option>
    <option value="Shopping">Shopping</option>
    <option value="Others">Others</option>
  </select>

  <select id="amountSizeCategory">
    <option value="">Select Amount Size</option>
    <option value="Big Amount">Big Amount</option>
    <option value="Small Amount">Small Amount</option>
  </select>

  <button onclick="addOrUpdateExpense()">Add Expense</button>

  <div class="search-filter">
    <input type="text" id="searchInput" placeholder="Search by name or category" oninput="renderExpenses()">
    <button onclick="exportCSV()">Export to CSV</button>
    <button onclick="resetAll()">Reset All</button>
  </div>

  <!-- Big Amount Section -->
  <div class="expense-section" id="bigAmountSection">
    <h3>Big Amount Expenses</h3>
    <div id="bigAmountList"></div>
    <div class="total" id="bigAmountTotal">Total: ₹0.00</div>
  </div>

  <!-- Small Amount Section -->
  <div class="expense-section" id="smallAmountSection">
    <h3>Small Amount Expenses</h3>
    <div id="smallAmountList"></div>
    <div class="total" id="smallAmountTotal">Total: ₹0.00</div>
  </div>

</div>

<script>
  let expenses = JSON.parse(localStorage.getItem('expenses')) || [];
  let editIndex = -1;

  const formatCurrency = (amount) => new Intl.NumberFormat('en-IN', {
    style: 'currency',
    currency: 'INR'
  }).format(amount);

  function saveExpenses() {
    localStorage.setItem('expenses', JSON.stringify(expenses));
  }

  function addOrUpdateExpense() {
    const name = document.getElementById('expenseName').value.trim();
    const amount = parseFloat(document.getElementById('expenseAmount').value);
    const category = document.getElementById('expenseCategory').value;
    const amountSize = document.getElementById('amountSizeCategory').value;
    const date = new Date().toISOString().split('T')[0];

    if (!name || isNaN(amount) || amount <= 0 || !category || !amountSize) {
      alert("Please enter valid name, amount, category, and select amount size.");
      return;
    }

    const newExpense = { name, amount, category, amountSize, date };

    if (editIndex >= 0) {
      expenses[editIndex] = newExpense;
      editIndex = -1;
    } else {
      expenses.push(newExpense);
    }

    saveExpenses();
    clearForm();
    renderExpenses();
  }

  function clearForm() {
    document.getElementById('expenseName').value = '';
    document.getElementById('expenseAmount').value = '';
    document.getElementById('expenseCategory').value = '';
    document.getElementById('amountSizeCategory').value = '';
  }

  function editExpense(index) {
    const exp = expenses[index];
    document.getElementById('expenseName').value = exp.name;
    document.getElementById('expenseAmount').value = exp.amount;
    document.getElementById('expenseCategory').value = exp.category;
    document.getElementById('amountSizeCategory').value = exp.amountSize;
    editIndex = index;
  }

  function deleteExpense(index) {
    if (confirm("Are you sure you want to delete this expense?")) {
      expenses.splice(index, 1);
      saveExpenses();
      renderExpenses();
    }
  }

  function renderExpenses() {
    const search = document.getElementById('searchInput').value.toLowerCase();

    const bigList = document.getElementById('bigAmountList');
    const smallList = document.getElementById('smallAmountList');
    bigList.innerHTML = '';
    smallList.innerHTML = '';

    let bigTotal = 0;
    let smallTotal = 0;

    expenses.forEach((exp, index) => {
      if (
        exp.name.toLowerCase().includes(search) ||
        exp.category.toLowerCase().includes(search) ||
        exp.amountSize.toLowerCase().includes(search)
      ) {
        const item = document.createElement('div');
        item.className = 'expense-item';
        item.innerHTML = `
          <div>
            <strong>${exp.name}</strong><br>
            ${formatCurrency(exp.amount)} | ${exp.category} 
            <span class="tag">${exp.amountSize}</span> | ${exp.date}
          </div>
          <div class="actions">
            <button onclick="editExpense(${index})">Edit</button>
            <button onclick="deleteExpense(${index})">Delete</button>
          </div>
        `;

        if (exp.amountSize === "Big Amount") {
          bigList.appendChild(item);
          bigTotal += exp.amount;
        } else if (exp.amountSize === "Small Amount") {
          smallList.appendChild(item);
          smallTotal += exp.amount;
        }
      }
    });

    document.getElementById('bigAmountTotal').textContent = `Total: ${formatCurrency(bigTotal)}`;
    document.getElementById('smallAmountTotal').textContent = `Total: ${formatCurrency(smallTotal)}`;
  }

  function exportCSV() {
    if (expenses.length === 0) {
      alert("No data to export.");
      return;
    }
    let csv = "Name,Amount,Category,Amount Size,Date\n";
    expenses.forEach(exp => {
      csv += `${exp.name},${exp.amount},${exp.category},${exp.amountSize},${exp.date}\n`;
    });

    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.setAttribute("href", url);
    link.setAttribute("download", "expenses.csv");
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  }

  function resetAll() {
    if (confirm("This will delete all expenses. Are you sure?")) {
      expenses = [];
      saveExpenses();
      renderExpenses();
    }
  }

  renderExpenses();
</script>

</body>
</html>
