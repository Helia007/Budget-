import { useState, useEffect, useRef } from "react";

const CAT_LABELS = {
  food: "🍵 Food & Drinks", transport: "🚌 Transport", shopping: "🛍️ Shopping",
  health: "🌸 Health & Beauty", entertainment: "🎬 Entertainment",
  utilities: "⚡ Utilities", rent: "🏠 Rent", other: "✨ Other",
};
const CAT_COLORS = {
  food: "#e07080", transport: "#6080d0", shopping: "#c050a0",
  health: "#50a870", entertainment: "#9060d0", utilities: "#b09030",
  rent: "#c06050", other: "#909090",
};
const CAT_BG = {
  food: "#fff0f0", transport: "#f0f4ff", shopping: "#fff0f8",
  health: "#f0fff4", entertainment: "#f8f0ff", utilities: "#fffbf0",
  rent: "#fff4f0", other: "#f4f0f4",
};

function todayStr() {
  return new Date().toISOString().split("T")[0];
}
function fmt(n) {
  return "€" + parseFloat(n || 0).toFixed(2);
}
function fmtDate(d) {
  return new Date(d + "T12:00:00").toLocaleDateString("en-IE", {
    day: "numeric", month: "short", year: "numeric",
  });
}
function weekStart() {
  const d = new Date();
  d.setHours(0, 0, 0, 0);
  d.setDate(d.getDate() - ((d.getDay() + 6) % 7));
  return d.toISOString().split("T")[0];
}
function monthStart() {
  const d = new Date();
  return d.getFullYear() + "-" + String(d.getMonth() + 1).padStart(2, "0") + "-01";
}
function yearStart() {
  return new Date().getFullYear() + "-01-01";
}
function lsGet(key, fallback) {
  try {
    const v = localStorage.getItem(key);
    return v !== null ? JSON.parse(v) : fallback;
  } catch (e) {
    return fallback;
  }
}
function lsSet(key, val) {
  try { localStorage.setItem(key, JSON.stringify(val)); } catch (e) {}
}

function exportToExcel(expenses, filter, catFilter) {
  var filtered = expenses.slice();
  if (filter === "week") filtered = filtered.filter(function(e) { return e.date >= weekStart(); });
  if (filter === "month") filtered = filtered.filter(function(e) { return e.date >= monthStart(); });
  if (filter === "year") filtered = filtered.filter(function(e) { return e.date >= yearStart(); });
  if (catFilter) filtered = filtered.filter(function(e) { return e.cat === catFilter; });
  if (!filtered.length) return "empty";
  var BOM = "\uFEFF";
  var header = "Date,Description,Category,Amount (EUR)";
  var rows = filtered.map(function(e) {
    var d = '"' + e.date + '"';
    var desc = '"' + String(e.desc).replace(/"/g, '""') + '"';
    var cat = '"' + CAT_LABELS[e.cat].replace(/[^\w\s&-]/g, "").trim() + '"';
    var amt = '"' + parseFloat(e.amount).toFixed(2) + '"';
    return [d, desc, cat, amt].join(",");
  });
  var csv = BOM + [header].concat(rows).join("\r\n");
  var blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
  var url = URL.createObjectURL(blob);
  var a = document.createElement("a");
  a.href = url;
  a.download = "rose-diary-" + todayStr() + ".csv";
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  setTimeout(function() { URL.revokeObjectURL(url); }, 2000);
  return "ok";
}

function Toast(props) {
  if (!props.msg) return null;
  return (
    <div style={{
      position: "fixed", bottom: 74, left: "50%", transform: "translateX(-50%)",
      background: "linear-gradient(135deg,#9e4e56,#c9727a)", color: "#fff",
      padding: "10px 22px", borderRadius: 30, fontSize: 13, fontWeight: 500,
      boxShadow: "0 8px 32px rgba(158,78,86,0.35)", zIndex: 9999,
      whiteSpace: "nowrap", pointerEvents: "none",
    }}>
      {props.msg}
    </div>
  );
}

function AccountModal(props) {
  var account = props.account;
  var currentVal = props.currentVal;
  var onClose = props.onClose;
  var onSave = props.onSave;
  var isRunning = account === "running";
  var exactRef = useRef("");
  var adjRef = useRef("");
  var [exact, setExact] = useState("");
  var [adj, setAdj] = useState("");

  return (
    <div
      onClick={function(e) { if (e.target === e.currentTarget) onClose(); }}
      style={{
        position: "fixed", inset: 0, background: "rgba(44,31,36,0.5)",
        zIndex: 500, display: "flex", alignItems: "center", justifyContent: "center",
        padding: 20, backdropFilter: "blur(4px)",
      }}
    >
      <div style={{
        background: "#fff", borderRadius: 24, padding: 28,
        width: "100%", maxWidth: 400,
        boxShadow: "0 20px 60px rgba(44,31,36,0.2)",
      }}>
        <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 24, fontWeight: 600, marginBottom: 4 }}>
          Edit {isRunning ? "Running" : "Savings"} Account
        </div>
        <div style={{ fontSize: 13, color: "#9a8690", marginBottom: 22 }}>
          Currently: <strong>{fmt(currentVal)}</strong>
        </div>

        <label style={{ fontSize: 10, textTransform: "uppercase", letterSpacing: 2, color: "#b07a8a", fontWeight: 600, display: "block", marginBottom: 6 }}>
          Set Exact Balance (€)
        </label>
        <input
          type="number" min="0" step="0.01" value={exact}
          onChange={function(e) { setExact(e.target.value); }}
          placeholder={parseFloat(currentVal).toFixed(2)}
          style={{ width: "100%", background: "#fdf0f1", border: "1.5px solid #e8d8db", borderRadius: 12, padding: "11px 14px", fontSize: 15, fontFamily: "'Jost',sans-serif", outline: "none", marginBottom: 10 }}
        />
        <button
          onClick={function() { if (exact !== "") onSave(parseFloat(exact), "set"); }}
          style={{ width: "100%", background: "linear-gradient(135deg,#9e4e56,#c9727a)", color: "#fff", border: "none", borderRadius: 12, padding: 12, fontSize: 14, fontWeight: 600, cursor: "pointer", fontFamily: "'Jost',sans-serif", marginBottom: 18 }}
        >
          Save Balance
        </button>

        <div style={{ height: 1, background: "#f2d6d9", marginBottom: 18 }} />

        <label style={{ fontSize: 10, textTransform: "uppercase", letterSpacing: 2, color: "#b07a8a", fontWeight: 600, display: "block", marginBottom: 8 }}>
          Quick Adjustment (€)
        </label>
        <input
          type="number" min="0" step="0.01" value={adj}
          onChange={function(e) { setAdj(e.target.value); }}
          placeholder="Amount to add or remove"
          style={{ width: "100%", background: "#fdf0f1", border: "1.5px solid #e8d8db", borderRadius: 12, padding: "11px 14px", fontSize: 15, fontFamily: "'Jost',sans-serif", outline: "none", marginBottom: 10 }}
        />
        <div style={{ display: "flex", gap: 10, marginBottom: 18 }}>
          <button
            onClick={function() { if (adj) onSave(parseFloat(adj), "+"); }}
            style={{ flex: 1, background: "#e2f4ec", color: "#308050", border: "1.5px solid #b7deca", borderRadius: 12, padding: 10, fontSize: 13, fontWeight: 600, cursor: "pointer", fontFamily: "'Jost',sans-serif" }}
          >＋ Add Funds</button>
          <button
            onClick={function() { if (adj) onSave(parseFloat(adj), "-"); }}
            style={{ flex: 1, background: "#fff0f1", color: "#9e4e56", border: "1.5px solid #f2d6d9", borderRadius: 12, padding: 10, fontSize: 13, fontWeight: 600, cursor: "pointer", fontFamily: "'Jost',sans-serif" }}
          >－ Reduce</button>
        </div>
        <button
          onClick={onClose}
          style={{ width: "100%", background: "#fdf0f1", color: "#9a8690", border: "1.5px solid #e8d8db", borderRadius: 12, padding: 10, fontSize: 13, cursor: "pointer", fontFamily: "'Jost',sans-serif" }}
        >Cancel</button>
      </div>
    </div>
  );
}

function ExpenseItem(props) {
  var e = props.e;
  var onDelete = props.onDelete;
  return (
    <div style={{
      background: "#fff", border: "1px solid #e8d8db", borderRadius: 15,
      padding: "12px 14px", display: "flex", alignItems: "center", gap: 10, marginBottom: 8,
    }}>
      <div style={{ width: 9, height: 9, borderRadius: "50%", background: CAT_COLORS[e.cat], flexShrink: 0 }} />
      <div style={{ flex: 1, fontSize: 13.5, fontWeight: 500, minWidth: 0, whiteSpace: "nowrap", overflow: "hidden", textOverflow: "ellipsis" }}>
        {e.desc}
      </div>
      <div style={{ display: "flex", flexDirection: "column", alignItems: "flex-end", gap: 2, flexShrink: 0 }}>
        <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 16, fontWeight: 600 }}>{fmt(e.amount)}</div>
        <span style={{ fontSize: 9, padding: "2px 7px", borderRadius: 20, fontWeight: 600, letterSpacing: 0.4, textTransform: "uppercase", background: CAT_BG[e.cat], color: CAT_COLORS[e.cat] }}>
          {CAT_LABELS[e.cat]}
        </span>
        <div style={{ fontSize: 9.5, color: "#9a8690" }}>{fmtDate(e.date)}</div>
      </div>
      <button
        onClick={function() { onDelete(e.id, e.amount); }}
        style={{ background: "none", border: "none", color: "#9a8690", cursor: "pointer", fontSize: 15, padding: 5, borderRadius: 8, flexShrink: 0 }}
      >🗑</button>
    </div>
  );
}

export default function App() {
  var [page, setPage] = useState("add");
  var [expenses, setExpenses] = useState(function() { return lsGet("rose_expenses", []); });
  var [running, setRunning] = useState(function() { return lsGet("rose_running", 0); });
  var [savings, setSavings] = useState(function() { return lsGet("rose_savings", 0); });
  var [modal, setModal] = useState(null);
  var [toast, setToast] = useState("");
  var [histFilter, setHistFilter] = useState("all");
  var [catFilter, setCatFilter] = useState("");
  var [desc, setDesc] = useState("");
  var [amount, setAmount] = useState("");
  var [cat, setCat] = useState("food");
  var [expDate, setExpDate] = useState(todayStr());
  var toastTimer = useRef(null);

  useEffect(function() { lsSet("rose_expenses", expenses); }, [expenses]);
  useEffect(function() { lsSet("rose_running", running); }, [running]);
  useEffect(function() { lsSet("rose_savings", savings); }, [savings]);

  function showToast(msg) {
    setToast(msg);
    clearTimeout(toastTimer.current);
    toastTimer.current = setTimeout(function() { setToast(""); }, 2800);
  }

  function addExpense() {
    var a = parseFloat(amount);
    if (!desc.trim() || isNaN(a) || a <= 0) { showToast("Fill in description & amount 🌸"); return; }
    if (a > running) { showToast("⚠️ Not enough in running account!"); return; }
    setExpenses(function(prev) {
      return [{ id: Date.now(), desc: desc.trim(), amount: a, cat: cat, date: expDate || todayStr() }].concat(prev);
    });
    setRunning(function(prev) { return parseFloat((prev - a).toFixed(2)); });
    setDesc(""); setAmount(""); setExpDate(todayStr());
    showToast("Added to your diary 💸");
  }

  function deleteExpense(id, amt) {
    setExpenses(function(prev) { return prev.filter(function(e) { return e.id !== id; }); });
    setRunning(function(prev) { return parseFloat((prev + amt).toFixed(2)); });
    showToast("Removed — refunded to running account ✓");
  }

  function handleAccountSave(val, op) {
    if (isNaN(val) || val < 0) { showToast("Invalid amount"); return; }
    var setter = modal === "running" ? setRunning : setSavings;
    var cur = modal === "running" ? running : savings;
    if (op === "set") setter(parseFloat(val.toFixed(2)));
    else if (op === "+") setter(parseFloat((cur + val).toFixed(2)));
    else if (op === "-") {
      if (cur - val < 0) { showToast("Balance can't go below €0"); return; }
      setter(parseFloat((cur - val).toFixed(2)));
    }
    setModal(null);
    showToast((modal === "running" ? "Running" : "Savings") + " account updated ✓");
  }

  function getFiltered() {
    var f = expenses.slice();
    if (histFilter === "week") f = f.filter(function(e) { return e.date >= weekStart(); });
    if (histFilter === "month") f = f.filter(function(e) { return e.date >= monthStart(); });
    if (histFilter === "year") f = f.filter(function(e) { return e.date >= yearStart(); });
    if (catFilter) f = f.filter(function(e) { return e.cat === catFilter; });
    return f;
  }

  function calcStreak() {
    var logged = {};
    expenses.forEach(function(e) { logged[e.date] = true; });
    var streak = 0;
    var d = new Date();
    var t = todayStr();
    if (!logged[t]) d.setDate(d.getDate() - 1);
    while (true) {
      var s = d.toISOString().split("T")[0];
      if (!logged[s]) break;
      streak++;
      d.setDate(d.getDate() - 1);
    }
    return streak;
  }

  var streak = calcStreak();
  var spentToday = expenses.filter(function(e) { return e.date === todayStr(); }).reduce(function(s, e) { return s + e.amount; }, 0);
  var spentMonth = expenses.filter(function(e) { return e.date >= monthStart(); }).reduce(function(s, e) { return s + e.amount; }, 0);
  var spentYear = expenses.filter(function(e) { return e.date >= yearStart(); }).reduce(function(s, e) { return s + e.amount; }, 0);
  var todayList = expenses.filter(function(e) { return e.date === todayStr(); });
  var filteredList = getFiltered();
  var monExp = expenses.filter(function(e) { return e.date >= monthStart(); });
  var catTotals = {};
  monExp.forEach(function(e) {
    if (!catTotals[e.cat]) catTotals[e.cat] = { total: 0, count: 0 };
    catTotals[e.cat].total += e.amount;
    catTotals[e.cat].count++;
  });
  var catSorted = Object.entries(catTotals).sort(function(a, b) { return b[1].total - a[1].total; });
  var loggedDates = {};
  expenses.forEach(function(e) { loggedDates[e.date] = true; });
  var last14 = [];
  for (var i = 13; i >= 0; i--) {
    var dd = new Date();
    dd.setDate(dd.getDate() - i);
    last14.push(dd.toISOString().split("T")[0]);
  }
  var allCats = {};
  expenses.forEach(function(e) { allCats[e.cat] = true; });
  var badges = [
    { id: "first", ico: "🌸", label: "First Entry", earned: expenses.length >= 1 },
    { id: "ten", ico: "💐", label: "10 Entries", earned: expenses.length >= 10 },
    { id: "fifty", ico: "🌹", label: "50 Entries", earned: expenses.length >= 50 },
    { id: "s3", ico: "🔥", label: "3-Day Streak", earned: streak >= 3 },
    { id: "s7", ico: "💫", label: "7-Day Streak", earned: streak >= 7 },
    { id: "s30", ico: "👑", label: "30-Day Streak", earned: streak >= 30 },
    { id: "cats", ico: "🎨", label: "All Categories", earned: Object.keys(allCats).length >= 8 },
    { id: "saver", ico: "💎", label: "Savings Set", earned: savings > 0 },
  ];
  var monthDaysLogged = {};
  expenses.filter(function(e) { return e.date >= monthStart(); }).forEach(function(e) { monthDaysLogged[e.date] = true; });
  var uniqueMonthDays = Object.keys(monthDaysLogged).length;

  var card = { background: "#fff", borderRadius: 20, padding: 20, border: "1px solid #e8d8db", boxShadow: "0 2px 20px rgba(180,100,110,0.08)", marginBottom: 16 };
  var labelStyle = { fontSize: 9.5, textTransform: "uppercase", letterSpacing: 2.5, color: "#9a8690", marginBottom: 8, display: "block" };
  var inputStyle = { background: "#fdf0f1", border: "1.5px solid #e8d8db", borderRadius: 12, padding: "11px 13px", fontSize: 14, color: "#2c1f24", fontFamily: "'Jost',sans-serif", outline: "none", width: "100%" };
  var selectStyle = { background: "#fdf0f1", border: "1.5px solid #e8d8db", borderRadius: 12, padding: "11px 13px", fontSize: 14, color: "#2c1f24", fontFamily: "'Jost',sans-serif", outline: "none", width: "100%" };

  function filterBtn(active) {
    return { background: active ? "#c9727a" : "#fff", color: active ? "#fff" : "#9a8690", border: "1.5px solid " + (active ? "#c9727a" : "#e8d8db"), borderRadius: 30, padding: "6px 13px", fontSize: 11.5, cursor: "pointer", fontFamily: "'Jost',sans-serif", fontWeight: 500 };
  }

  return (
    <div style={{ fontFamily: "'Jost',sans-serif", background: "#fdf0f1", minHeight: "100vh", color: "#2c1f24" }}>
      <style>{`* { box-sizing: border-box; } input[type=number]::-webkit-inner-spin-button { opacity: 0.4; }`}</style>

      {/* TOPBAR */}
      <div style={{ background: "linear-gradient(135deg,#9e4e56 0%,#c9727a 60%,#b07a8a 100%)", padding: "0 18px", height: 54, display: "flex", alignItems: "center", justifyContent: "space-between", position: "sticky", top: 0, zIndex: 200, boxShadow: "0 2px 20px rgba(150,60,70,0.25)" }}>
        <div style={{ fontFamily: "'Cormorant Garamond',serif", color: "#fff", fontSize: 24, fontWeight: 600, letterSpacing: 1 }}>
          Rosé <span style={{ color: "rgba(255,255,255,0.6)", fontSize: 9, letterSpacing: 3, textTransform: "uppercase" }}>money diary</span>
        </div>
        <div style={{ color: "rgba(255,255,255,0.8)", fontSize: 11 }}>
          {streak > 0 ? "🔥 " + streak + "d streak" : ""}
        </div>
      </div>

      {/* ADD PAGE */}
      {page === "add" && (
        <div style={{ padding: "18px 16px 82px", maxWidth: 780, margin: "0 auto" }}>
          <div style={{ marginBottom: 18 }}>
            <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 30, fontWeight: 600, marginBottom: 3 }}>
              Add <em style={{ color: "#c9727a", fontStyle: "italic" }}>Expense</em>
            </div>
            <div style={{ fontSize: 12.5, color: "#9a8690" }}>Logged from your running account.</div>
          </div>

          {/* ACCOUNTS */}
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 13, marginBottom: 13 }}>
            <div onClick={function() { setModal("running"); }} style={{ background: "linear-gradient(135deg,#fff5f6,#fdeef0)", borderRadius: 20, padding: 16, border: "1px solid #f2d6d9", cursor: "pointer" }}>
              <div style={{ fontSize: 20, marginBottom: 5 }}>👛</div>
              <div style={{ fontSize: 9, textTransform: "uppercase", letterSpacing: 2, color: "#9a8690", marginBottom: 3 }}>Running Account</div>
              <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 24, fontWeight: 600, color: "#c9727a", lineHeight: 1 }}>{fmt(running)}</div>
              <div style={{ fontSize: 10, color: "#9a8690", marginTop: 3 }}>{fmt(spentMonth)} spent this month</div>
              <div style={{ fontSize: 9, color: "#9a8690", marginTop: 5, opacity: 0.7 }}>Tap to edit ✏️</div>
            </div>
            <div onClick={function() { setModal("savings"); }} style={{ background: "linear-gradient(135deg,#f0fff8,#e2f4ec)", borderRadius: 20, padding: 16, border: "1px solid #b7deca", cursor: "pointer" }}>
              <div style={{ fontSize: 20, marginBottom: 5 }}>🏦</div>
              <div style={{ fontSize: 9, textTransform: "uppercase", letterSpacing: 2, color: "#9a8690", marginBottom: 3 }}>Savings Account</div>
              <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 24, fontWeight: 600, color: "#6aab8e", lineHeight: 1 }}>{fmt(savings)}</div>
              <div style={{ fontSize: 10, color: "#9a8690", marginTop: 3 }}>Safe & untouched</div>
              <div style={{ fontSize: 9, color: "#9a8690", marginTop: 5, opacity: 0.7 }}>Tap to edit ✏️</div>
            </div>
          </div>

          {/* AVAILABLE BANNER */}
          <div style={{
            background: running <= 0 ? "linear-gradient(135deg,#7a2020,#c04040)" : running < 50 ? "linear-gradient(135deg,#8a6020,#c9a040)" : "linear-gradient(135deg,#9e4e56,#c9727a)",
            borderRadius: 18, padding: "14px 18px", marginBottom: 16,
            display: "flex", alignItems: "center", justifyContent: "space-between",
            boxShadow: "0 4px 20px rgba(158,78,86,0.28)",
          }}>
            <div>
              <div style={{ color: "rgba(255,255,255,0.7)", fontSize: 10, textTransform: "uppercase", letterSpacing: 2, marginBottom: 3 }}>Available to Spend</div>
              <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 30, color: running < 50 ? "#ffe08a" : "#fff", fontWeight: 600, lineHeight: 1 }}>{fmt(running)}</div>
              {running < 50 && running > 0 && <div style={{ fontSize: 10, color: "rgba(255,255,255,0.7)", marginTop: 3 }}>Running low — top up soon!</div>}
              {running <= 0 && <div style={{ fontSize: 10, color: "rgba(255,255,255,0.8)", marginTop: 3 }}>No funds available</div>}
            </div>
            <div style={{ fontSize: 34, opacity: 0.8 }}>💳</div>
          </div>

          {/* FORM */}
          <div style={card}>
            <div style={{ display: "flex", flexDirection: "column", gap: 13 }}>
              <div>
                <label style={labelStyle}>Description</label>
                <input style={inputStyle} value={desc} onChange={function(e) { setDesc(e.target.value); }} placeholder="e.g. Matcha latte, new earrings…" onKeyDown={function(e) { if (e.key === "Enter") addExpense(); }} />
              </div>
              <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12 }}>
                <div>
                  <label style={labelStyle}>Amount (€)</label>
                  <input style={inputStyle} type="number" min="0" step="0.01" value={amount} onChange={function(e) { setAmount(e.target.value); }} placeholder="0.00" onKeyDown={function(e) { if (e.key === "Enter") addExpense(); }} />
                </div>
                <div>
                  <label style={labelStyle}>Date</label>
                  <input style={inputStyle} type="date" value={expDate} onChange={function(e) { setExpDate(e.target.value); }} />
                </div>
              </div>
              <div>
                <label style={labelStyle}>Category</label>
                <select style={selectStyle} value={cat} onChange={function(e) { setCat(e.target.value); }}>
                  {Object.entries(CAT_LABELS).map(function(entry) { return <option key={entry[0]} value={entry[0]}>{entry[1]}</option>; })}
                </select>
              </div>
              <button
                style={{ background: "linear-gradient(135deg,#9e4e56,#c9727a)", color: "#fff", border: "none", borderRadius: 14, padding: 13, fontSize: 14, fontWeight: 600, cursor: "pointer", fontFamily: "'Jost',sans-serif", width: "100%", marginTop: 4 }}
                onClick={addExpense}
              >＋ Add to Diary</button>
            </div>
          </div>

          {/* TODAY LIST */}
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 12 }}>
            <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 21, fontWeight: 600 }}>Today's Log</div>
            <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 22, color: "#c9727a", fontWeight: 600 }}>{fmt(spentToday)}</div>
          </div>
          {todayList.length === 0
            ? <div style={{ textAlign: "center", padding: "36px 20px", color: "#9a8690" }}><div style={{ fontSize: 38, marginBottom: 10 }}>🌸</div><div>No entries today yet!</div></div>
            : todayList.map(function(e) { return <ExpenseItem key={e.id} e={e} onDelete={deleteExpense} />; })
          }
        </div>
      )}

      {/* OVERVIEW PAGE */}
      {page === "overview" && (
        <div style={{ padding: "18px 16px 82px", maxWidth: 780, margin: "0 auto" }}>
          <div style={{ marginBottom: 18 }}>
            <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 30, fontWeight: 600, marginBottom: 3 }}>My <em style={{ color: "#c9727a", fontStyle: "italic" }}>Overview</em></div>
            <div style={{ fontSize: 12.5, color: "#9a8690" }}>How you're doing across all periods.</div>
          </div>
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 13, marginBottom: 13 }}>
            <div style={{ background: "linear-gradient(135deg,#fff5f6,#fdeef0)", borderRadius: 18, padding: 15, border: "1px solid #f2d6d9" }}>
              <div style={{ fontSize: 9, textTransform: "uppercase", letterSpacing: 2, color: "#9a8690", marginBottom: 3 }}>👛 Running</div>
              <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 22, color: "#c9727a", fontWeight: 600 }}>{fmt(running)}</div>
              <div style={{ fontSize: 10, color: "#9a8690", marginTop: 2 }}>balance remaining</div>
            </div>
            <div style={{ background: "linear-gradient(135deg,#f0fff8,#e2f4ec)", borderRadius: 18, padding: 15, border: "1px solid #b7deca" }}>
              <div style={{ fontSize: 9, textTransform: "uppercase", letterSpacing: 2, color: "#9a8690", marginBottom: 3 }}>🏦 Savings</div>
              <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 22, color: "#6aab8e", fontWeight: 600 }}>{fmt(savings)}</div>
              <div style={{ fontSize: 10, color: "#9a8690", marginTop: 2 }}>safe & growing</div>
            </div>
          </div>
          <div style={{ display: "grid", gridTemplateColumns: "repeat(3,1fr)", gap: 11, marginBottom: 14 }}>
            {[
              { label: "Today", val: fmt(spentToday), count: todayList.length, color: "#c9727a" },
              { label: "This Month", val: fmt(spentMonth), count: monExp.length, color: "#c9a96e" },
              { label: "This Year", val: fmt(spentYear), count: expenses.filter(function(e) { return e.date >= yearStart(); }).length, color: "#b07a8a" },
            ].map(function(s) {
              return (
                <div key={s.label} style={{ background: "#fff", borderRadius: 16, padding: "15px 11px", border: "1px solid #e8d8db", textAlign: "center" }}>
                  <div style={{ fontSize: 9, textTransform: "uppercase", letterSpacing: 2, color: "#9a8690", marginBottom: 5 }}>{s.label}</div>
                  <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 20, fontWeight: 600, color: s.color }}>{s.val}</div>
                  <div style={{ fontSize: 9.5, color: "#9a8690", marginTop: 3 }}>{s.count} items</div>
                </div>
              );
            })}
          </div>
          <div style={card}>
            <div style={labelStyle}>Top Categories — This Month</div>
            {catSorted.length === 0
              ? <div style={{ textAlign: "center", color: "#9a8690", padding: 16, fontSize: 13 }}>No data this month yet 🌸</div>
              : (
                <table style={{ width: "100%", borderCollapse: "collapse" }}>
                  <thead><tr>{["Category", "Spent", "#"].map(function(h) { return <th key={h} style={{ textAlign: "left", fontSize: 9, textTransform: "uppercase", letterSpacing: 2, color: "#9a8690", padding: "7px 9px", borderBottom: "1px solid #e8d8db" }}>{h}</th>; })}</tr></thead>
                  <tbody>
                    {catSorted.map(function(entry) {
                      var c = entry[0]; var d = entry[1];
                      return (
                        <tr key={c}>
                          <td style={{ padding: "9px", borderBottom: "1px solid #fdf0f1" }}><span style={{ fontSize: 9, padding: "2px 7px", borderRadius: 20, fontWeight: 600, textTransform: "uppercase", background: CAT_BG[c], color: CAT_COLORS[c] }}>{CAT_LABELS[c]}</span></td>
                          <td style={{ padding: "9px", borderBottom: "1px solid #fdf0f1", fontWeight: 600, fontSize: 13 }}>{fmt(d.total)}</td>
                          <td style={{ padding: "9px", borderBottom: "1px solid #fdf0f1", color: "#9a8690", fontSize: 13 }}>{d.count}</td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              )
            }
          </div>
        </div>
      )}

      {/* HISTORY PAGE */}
      {page === "history" && (
        <div style={{ padding: "18px 16px 82px", maxWidth: 780, margin: "0 auto" }}>
          <div style={{ marginBottom: 18 }}>
            <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 30, fontWeight: 600, marginBottom: 3 }}>Spending <em style={{ color: "#c9727a", fontStyle: "italic" }}>History</em></div>
            <div style={{ fontSize: 12.5, color: "#9a8690" }}>Every penny, accounted for.</div>
          </div>
          <div style={{ display: "flex", gap: 7, marginBottom: 11, flexWrap: "wrap" }}>
            {[["all", "All"], ["week", "This Week"], ["month", "This Month"], ["year", "This Year"]].map(function(entry) {
              return <button key={entry[0]} style={filterBtn(histFilter === entry[0])} onClick={function() { setHistFilter(entry[0]); }}>{entry[1]}</button>;
            })}
          </div>
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 13, flexWrap: "wrap", gap: 8 }}>
            <select style={{ background: "#fff", border: "1.5px solid #e8d8db", borderRadius: 30, padding: "6px 13px", fontSize: 11.5, fontFamily: "'Jost',sans-serif", color: "#2c1f24", outline: "none", cursor: "pointer" }} value={catFilter} onChange={function(e) { setCatFilter(e.target.value); }}>
              <option value="">All Categories</option>
              {Object.entries(CAT_LABELS).map(function(entry) { return <option key={entry[0]} value={entry[0]}>{entry[1]}</option>; })}
            </select>
            <button
              onClick={function() { var res = exportToExcel(expenses, histFilter, catFilter); showToast(res === "empty" ? "No data to export!" : "✅ Exported! Open in Excel 📊"); }}
              style={{ background: "linear-gradient(135deg,#c9a96e,#b8955a)", color: "#fff", border: "none", borderRadius: 10, padding: "8px 15px", fontSize: 11.5, cursor: "pointer", fontFamily: "'Jost',sans-serif", fontWeight: 600 }}
            >📥 Export to Excel</button>
          </div>
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 12 }}>
            <div style={{ fontSize: 12, color: "#9a8690" }}>{filteredList.length} transactions</div>
            <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 18, color: "#c9727a", fontWeight: 600 }}>Total: {fmt(filteredList.reduce(function(s, e) { return s + e.amount; }, 0))}</div>
          </div>
          {filteredList.length === 0
            ? <div style={{ textAlign: "center", padding: "40px 20px", color: "#9a8690" }}><div style={{ fontSize: 38, marginBottom: 10 }}>🔍</div><div>Nothing found.</div></div>
            : filteredList.map(function(e) { return <ExpenseItem key={e.id} e={e} onDelete={deleteExpense} />; })
          }
        </div>
      )}

      {/* STREAK PAGE */}
      {page === "streak" && (
        <div style={{ padding: "18px 16px 82px", maxWidth: 780, margin: "0 auto" }}>
          <div style={{ marginBottom: 18 }}>
            <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 30, fontWeight: 600, marginBottom: 3 }}>My <em style={{ color: "#c9727a", fontStyle: "italic" }}>Streak</em></div>
            <div style={{ fontSize: 12.5, color: "#9a8690" }}>Log every day and grow your flame 🔥</div>
          </div>
          <div style={{ background: "linear-gradient(135deg,#fff5f6,#fdeef0)", border: "1.5px solid #f2d6d9", borderRadius: 20, padding: 18, marginBottom: 16, display: "flex", alignItems: "center", gap: 14 }}>
            <div style={{ fontSize: 38 }}>{streak >= 7 ? "🔥" : streak >= 3 ? "🕯️" : "💧"}</div>
            <div style={{ flex: 1 }}>
              <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 19, fontWeight: 600 }}>Logging Streak</div>
              <div style={{ fontSize: 11, color: "#9a8690", marginTop: 2 }}>Log an expense every day!</div>
            </div>
            <div style={{ textAlign: "center" }}>
              <div style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 36, color: "#c9727a", fontWeight: 600, lineHeight: 1 }}>{streak}</div>
              <div style={{ fontSize: 9, color: "#9a8690", textTransform: "uppercase", letterSpacing: 2 }}>days</div>
            </div>
          </div>
          <div style={card}>
            <div style={labelStyle}>Last 14 Days</div>
            <div style={{ display: "flex", gap: 5, flexWrap: "wrap" }}>
              {last14.map(function(d) {
                var isL = !!loggedDates[d]; var isT = d === todayStr();
                var lbl = new Date(d + "T12:00:00").toLocaleDateString("en-IE", { weekday: "narrow" });
                return (
                  <div key={d} title={fmtDate(d)} style={{ width: 34, height: 34, borderRadius: "50%", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 9, fontWeight: 600, border: isT ? "2.5px solid #c9727a" : "1.5px solid #e8d8db", flexDirection: "column", color: isL ? "#fff" : "#9a8690", background: isL ? "linear-gradient(135deg,#9e4e56,#c9727a)" : "#fdf0f1", boxShadow: isL ? "0 2px 8px rgba(201,114,122,0.3)" : "none" }}>
                    <span style={{ fontSize: 8, opacity: 0.8 }}>{lbl}</span>
                    <span>{isL ? "✓" : "·"}</span>
                  </div>
                );
              })}
            </div>
          </div>
          <div style={card}>
            <div style={labelStyle}>Achievements 🏆</div>
            <div style={{ display: "flex", gap: 9, flexWrap: "wrap" }}>
              {badges.map(function(b) {
                return (
                  <div key={b.id} style={{ display: "flex", flexDirection: "column", alignItems: "center", gap: 3, background: b.earned ? "linear-gradient(135deg,#f5edd8,#fdf0d0)" : "#fff", border: "1.5px solid " + (b.earned ? "#c9a96e" : "#e8d8db"), borderRadius: 14, padding: "9px 12px", fontSize: 9, color: b.earned ? "#c9a96e" : "#9a8690", textAlign: "center", textTransform: "uppercase", minWidth: 66 }}>
                    <div style={{ fontSize: 22 }}>{b.earned ? b.ico : "🔒"}</div>
                    <div>{b.label}</div>
                  </div>
                );
              })}
            </div>
          </div>
          <div style={{ ...card, background: "linear-gradient(135deg,#fff5f6,#fdf0f2)" }}>
            <div style={labelStyle}>Monthly Challenge</div>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "baseline", marginBottom: 7 }}>
              <span style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 15 }}>Log 20 days this month</span>
              <span style={{ fontFamily: "'Cormorant Garamond',serif", fontSize: 17, color: "#c9727a" }}>{uniqueMonthDays}/20</span>
            </div>
            <div style={{ height: 7, background: "#f2d6d9", borderRadius: 8, overflow: "hidden" }}>
              <div style={{ height: "100%", background: "linear-gradient(90deg,#9e4e56,#c9727a)", borderRadius: 8, width: Math.min(100, Math.round(uniqueMonthDays / 20 * 100)) + "%", transition: "width .6s ease" }} />
            </div>
            <div style={{ fontSize: 10, color: "#9a8690", marginTop: 5 }}>
              {uniqueMonthDays >= 20 ? "🏆 Challenge complete!" : uniqueMonthDays >= 15 ? "Almost there 💕" : uniqueMonthDays >= 10 ? "Halfway! Keep going ✨" : "Every entry counts 🌸"}
            </div>
          </div>
        </div>
      )}

      {/* BOTTOM NAV */}
      <nav style={{ position: "fixed", bottom: 0, left: 0, right: 0, background: "#fff", borderTop: "1px solid #e8d8db", display: "flex", zIndex: 200, boxShadow: "0 -2px 16px rgba(180,100,110,0.1)", height: 62 }}>
        {[
          { id: "add", ico: "✏️", label: "Add" },
          { id: "overview", ico: "📊", label: "Overview" },
          { id: "history", ico: "📋", label: "History" },
          { id: "streak", ico: "🔥", label: "Streak" },
        ].map(function(n) {
          return (
            <button key={n.id} onClick={function() { setPage(n.id); }} style={{ flex: 1, display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", gap: 2, background: "none", border: "none", borderTop: page === n.id ? "2.5px solid #c9727a" : "2.5px solid transparent", cursor: "pointer", fontFamily: "'Jost',sans-serif", color: page === n.id ? "#c9727a" : "#9a8690", fontSize: 9.5, fontWeight: 500, padding: "6px 2px", letterSpacing: 0.5, textTransform: "uppercase" }}>
              <span style={{ fontSize: 19 }}>{n.ico}</span>{n.label}
            </button>
          );
        })}
      </nav>

      {modal && (
        <AccountModal
          account={modal}
          currentVal={modal === "running" ? running : savings}
          onClose={function() { setModal(null); }}
          onSave={handleAccountSave}
        />
      )}

      <Toast msg={toast} />
    </div>
  );
}
