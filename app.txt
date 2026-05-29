import { useState } from "react";
import axios from "axios";
import "./App.css";
import Landing from "./Landing";

function App() {
  const [entered, setEntered] = useState(false);

  const [id, setId] = useState("");
  const [amount, setAmount] = useState("");
  const [account, setAccount] = useState(null);

  const BASE = "http://localhost:3001";

  
  if (!entered) {
    return <Landing enterApp={() => setEntered(true)} />;
  }


  const getAccount = async () => {
    try {
      const res = await axios.get(`${BASE}/account/${id}`);
      if (res.data.message) {
        alert(res.data.message);
        setAccount(null);
      } else {
        setAccount(res.data);
      }
    } catch {
      alert("Server error ❌");
    }
  };

  
  const deposit = async () => {
    try {
      const res = await axios.post(`${BASE}/deposit`, {
        account_id: Number(id),
        amount: Number(amount),
      });

      alert(res.data.message);
      getAccount();
    } catch {
      alert("Deposit failed ❌");
    }
  };

  
  const withdraw = async () => {
    try {
      const res = await axios.post(`${BASE}/withdraw`, {
        account_id: Number(id),
        amount: Number(amount),
      });

      alert(res.data.message);
      getAccount();
    } catch {
      alert("Withdraw failed ❌");
    }
  };

  return (
    <div className="container">
      <h1 style={{color: "black"}}>🏦 DigiBank</h1>

      <div className="card">
        <input
          placeholder="Enter Account ID"
          value={id}
          onChange={(e) => setId(e.target.value)}
        />

        <button onClick={getAccount}>Search</button>

        <input
          placeholder="Enter Amount"
          value={amount}
          onChange={(e) => setAmount(e.target.value)}
        />

        <div className="btns">
          <button className="deposit" onClick={deposit}>
            Deposit
          </button>

          <button className="withdraw" onClick={withdraw}>
            Withdraw
          </button>
        </div>

        {account && (
          <div className="details">
            <h3>Account Details</h3>
            <p><b>ID:</b> {account.account_id}</p>
            <p><b>Name:</b> {account.name}</p>
            <p><b>Type:</b> {account.account_type}</p>
            <p><b>Balance:</b> ₹{account.balance}</p>
          </div>
        )}
      </div>
    </div>
  );
}

export default App;