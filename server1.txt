const express = require("express");
const mysql = require("mysql2");
const cors = require("cors");

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// 🔗 MySQL Connection
const db = mysql.createConnection({
  host: "localhost",
  user: "root",
  password: "ammu1540", // your MySQL password
  database: "bank_db", // your DB name
});

db.connect((err) => {
  if (err) {
    console.log("DB ERROR:", err);
  } else {
    console.log("MySQL Connected ✅");
  }
});

// =========================
// ✅ GET ACCOUNT DETAILS
// =========================
app.get("/account/:id", (req, res) => {
  const id = req.params.id;

  db.query(
    `SELECT account.*, customers.name 
     FROM account 
     JOIN customers 
     ON account.customer_id = customers.customer_id 
     WHERE account.account_id = ?`,
    [id],
    (err, result) => {
      if (err) {
        console.log("ERROR:", err);
        return res.status(500).json({ message: "Server error" });
      }

      if (result.length === 0) {
        return res.json({ message: "Account not found" });
      }

      res.json(result[0]);
    }
  );
});

// =========================
// 💰 DEPOSIT
// =========================
app.post("/deposit", (req, res) => {
  const { account_id, amount } = req.body;

  // 1. Update balance
  db.query(
    "UPDATE account SET balance = balance + ? WHERE account_id = ?",
    [amount, account_id],
    (err) => {
      if (err) {
        console.log(err);
        return res.json({ message: "Deposit failed ❌" });
      }

      // 2. Insert into transaction table
      db.query(
        "INSERT INTO transaction (account_id, type, amount) VALUES (?, 'DEPOSIT', ?)",
        [account_id, amount],
        (err2) => {
          if (err2) {
            console.log(err2);
            return res.json({ message: "Transaction not recorded ❌" });
          }

          res.json({ message: "Deposit successful ✅" });
        }
      );
    }
  );
});
// =========================
// 💸 WITHDRAW
// =========================
app.post("/withdraw", (req, res) => {
  const { account_id, amount } = req.body;

  // 1. Check balance
  db.query(
    "SELECT balance FROM account WHERE account_id = ?",
    [account_id],
    (err, result) => {
      if (err) return res.json({ message: "Error ❌" });

      if (result[0].balance < amount) {
        return res.json({ message: "Insufficient balance ❌" });
      }

      // 2. Update balance
      db.query(
        "UPDATE account SET balance = balance - ? WHERE account_id = ?",
        [amount, account_id],
        (err2) => {
          if (err2) return res.json({ message: "Withdraw failed ❌" });

          // 3. Insert transaction
          db.query(
            "INSERT INTO transaction (account_id, type, amount) VALUES (?, 'WITHDRAW', ?)",
            [account_id, amount],
            (err3) => {
              if (err3) {
                console.log(err3);
                return res.json({ message: "Transaction not recorded ❌" });
              }

              res.json({ message: "Withdraw successful ✅" });
            }
          );
        }
      );
    }
  );
});

const PORT=3001;
app.listen(PORT, () => {
  console.log("Server running on port🚀" + PORT);
});