import { useState, useEffect, useCallback } from "react";
import {
  BarChart, Bar, XAxis, YAxis, CartesianGrid,
  Tooltip, ResponsiveContainer, Legend,
} from "recharts";

const API = process.env.REACT_APP_API_URL || "http://localhost:5000";

const generateTimes = () => {
  const times = [];
  for (let h = 5; h < 24; h++) {
    ["00", "30"].forEach(m => {
      const hour12 = h > 12 ? h - 12 : h === 0 ? 12 : h;
      const ampm = h < 12 ? "AM" : "PM";
      const label = `${hour12}:${m} ${ampm}`;
      const value = `${String(h).padStart(2, "0")}:${m}`;
      times.push({ label, value });
    });
  }
  times.push({ label: "12:00 AM", value: "24:00" });
  return times;
};
const TIME_OPTIONS = generateTimes();
const today = () => new Date().toISOString().split("T")[0];

/* ── CSS injected globally ── */
const globalCSS = `
  @import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;900&family=DM+Sans:wght@300;400;500;600&display=swap');
  @import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css');

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --gold: #c9a84c;
    --gold2: #f0d080;
    --dark: #080c14;
    --dark2: #0d1525;
    --dark3: #111927;
    --card: rgba(255,255,255,0.035);
    --border: rgba(201,168,76,0.18);
    --text: #e8dcc8;
    --muted: #6b7280;
    --green: #22c55e;
    --red: #ef4444;
    --blue: #3b82f6;
  }

  html { scroll-behavior: smooth; }

  body {
    background: var(--dark);
    color: var(--text);
    font-family: 'DM Sans', sans-serif;
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* Animated background */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background:
      radial-gradient(ellipse 80% 50% at 20% 10%, rgba(201,168,76,0.06) 0%, transparent 60%),
      radial-gradient(ellipse 60% 40% at 80% 90%, rgba(59,130,246,0.05) 0%, transparent 60%),
      radial-gradient(ellipse 40% 60% at 60% 40%, rgba(201,168,76,0.03) 0%, transparent 50%);
    pointer-events: none;
    z-index: 0;
  }

  /* Particle dots */
  body::after {
    content: '';
    position: fixed;
    inset: 0;
    background-image:
      radial-gradient(circle, rgba(201,168,76,0.15) 1px, transparent 1px);
    background-size: 60px 60px;
    pointer-events: none;
    z-index: 0;
    opacity: 0.4;
    animation: drift 20s linear infinite;
  }

  @keyframes drift {
    0%   { transform: translate(0,0); }
    100% { transform: translate(60px, 60px); }
  }

  /* Animations */
  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(30px); }
    to   { opacity: 1; transform: translateY(0); }
  }
  @keyframes fadeIn {
    from { opacity: 0; }
    to   { opacity: 1; }
  }
  @keyframes shimmer {
    0%   { background-position: -200% center; }
    100% { background-position: 200% center; }
  }
  @keyframes pulse-gold {
    0%, 100% { box-shadow: 0 0 0 0 rgba(201,168,76,0.4); }
    50%       { box-shadow: 0 0 0 8px rgba(201,168,76,0); }
  }
  @keyframes spin-slow {
    from { transform: rotate(0deg); }
    to   { transform: rotate(360deg); }
  }
  @keyframes bounce-in {
    0%   { transform: scale(0.3); opacity: 0; }
    50%  { transform: scale(1.05); }
    70%  { transform: scale(0.95); }
    100% { transform: scale(1); opacity: 1; }
  }
  @keyframes slide-down {
    from { transform: translateY(-100%); opacity: 0; }
    to   { transform: translateY(0); opacity: 1; }
  }
  @keyframes glow {
    0%, 100% { text-shadow: 0 0 10px rgba(201,168,76,0.3); }
    50%       { text-shadow: 0 0 25px rgba(201,168,76,0.7), 0 0 50px rgba(201,168,76,0.3); }
  }

  .animate-fadeup  { animation: fadeUp 0.6s ease both; }
  .animate-fadein  { animation: fadeIn 0.4s ease both; }
  .animate-bouncein { animation: bounce-in 0.5s cubic-bezier(0.36,0.07,0.19,0.97) both; }

  .delay-1 { animation-delay: 0.1s; }
  .delay-2 { animation-delay: 0.2s; }
  .delay-3 { animation-delay: 0.3s; }
  .delay-4 { animation-delay: 0.4s; }
  .delay-5 { animation-delay: 0.5s; }

  /* Nav */
  .nav-premium {
    position: sticky; top: 0; z-index: 1000;
    background: rgba(8,12,20,0.92);
    border-bottom: 1px solid var(--border);
    backdrop-filter: blur(20px);
    animation: slide-down 0.5s ease;
  }
  .nav-inner {
    max-width: 960px; margin: 0 auto;
    padding: 0 20px; height: 60px;
    display: flex; align-items: center; justify-content: space-between;
  }
  .nav-logo {
    font-family: 'Playfair Display', serif;
    font-size: 20px; font-weight: 900;
    background: linear-gradient(135deg, var(--gold), var(--gold2), var(--gold));
    background-size: 200% auto;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    animation: shimmer 3s linear infinite;
    display: flex; align-items: center; gap: 10px;
  }
  .nav-links { display: flex; align-items: center; gap: 6px; }
  .nav-btn {
    background: transparent; border: 1px solid transparent;
    color: var(--muted); font-family: 'DM Sans', sans-serif;
    font-size: 13px; font-weight: 500;
    padding: 7px 16px; border-radius: 8px;
    cursor: pointer; transition: all 0.25s;
    display: flex; align-items: center; gap: 7px;
  }
  .nav-btn:hover { color: var(--gold); border-color: var(--border); }
  .nav-btn.active {
    color: var(--dark); font-weight: 700;
    background: linear-gradient(135deg, var(--gold), var(--gold2));
    border-color: transparent;
  }
  .nav-divider { color: rgba(201,168,76,0.3); font-size: 18px; }

  /* Hero */
  .hero-section {
    text-align: center; padding: 60px 20px 40px;
    position: relative; z-index: 1;
  }
  .hero-badge {
    display: inline-flex; align-items: center; gap: 8px;
    background: rgba(201,168,76,0.08);
    border: 1px solid rgba(201,168,76,0.25);
    border-radius: 50px; padding: 6px 18px;
    font-size: 11px; font-weight: 600;
    color: var(--gold); letter-spacing: 2px;
    text-transform: uppercase; margin-bottom: 20px;
  }
  .hero-title {
    font-family: 'Playfair Display', serif;
    font-size: clamp(36px, 7vw, 64px);
    font-weight: 900; line-height: 1.1;
    background: linear-gradient(135deg, #fff 0%, var(--gold2) 50%, var(--gold) 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    animation: glow 3s ease-in-out infinite;
    margin-bottom: 16px;
  }
  .hero-sub {
    font-size: 15px; color: var(--muted);
    max-width: 400px; margin: 0 auto 32px;
    line-height: 1.7;
  }
  .hero-stats {
    display: flex; justify-content: center; gap: 40px;
    margin-top: 32px; flex-wrap: wrap;
  }
  .hero-stat { text-align: center; }
  .hero-stat-num {
    font-family: 'Playfair Display', serif;
    font-size: 28px; font-weight: 900;
    color: var(--gold);
  }
  .hero-stat-label { font-size: 11px; color: var(--muted); letter-spacing: 1px; text-transform: uppercase; margin-top: 2px; }

  /* Cards */
  .card-premium {
    background: var(--card);
    border: 1px solid var(--border);
    border-radius: 20px;
    padding: 28px;
    backdrop-filter: blur(12px);
    position: relative; z-index: 1;
    transition: border-color 0.3s, transform 0.3s;
  }
  .card-premium:hover { border-color: rgba(201,168,76,0.35); }
  .card-premium::before {
    content: '';
    position: absolute; inset: 0;
    border-radius: 20px;
    background: linear-gradient(135deg, rgba(201,168,76,0.04) 0%, transparent 60%);
    pointer-events: none;
  }
  .card-title {
    font-family: 'Playfair Display', serif;
    font-size: 22px; font-weight: 700;
    color: #fff; margin-bottom: 4px;
    display: flex; align-items: center; gap: 10px;
  }
  .card-sub { font-size: 13px; color: var(--muted); margin-bottom: 20px; }

  /* Form */
  .form-field { display: flex; flex-direction: column; gap: 7px; }
  .form-label {
    font-size: 11px; font-weight: 600;
    color: var(--gold); letter-spacing: 1.5px;
    text-transform: uppercase;
    display: flex; align-items: center; gap: 6px;
  }
  .form-input {
    padding: 13px 16px;
    border-radius: 12px;
    border: 1px solid rgba(201,168,76,0.15);
    background: rgba(255,255,255,0.04);
    color: #f1f5f9; font-family: 'DM Sans', sans-serif;
    font-size: 14px; outline: none;
    width: 100%; transition: all 0.25s;
  }
  .form-input:focus {
    border-color: var(--gold);
    background: rgba(201,168,76,0.06);
    box-shadow: 0 0 0 3px rgba(201,168,76,0.1);
  }
  .form-input::placeholder { color: #374151; }
  .form-input option { background: #1a2035; }

  /* Time row */
  .time-row { display: flex; gap: 12px; align-items: flex-end; }
  .time-sep {
    color: var(--gold); font-size: 20px;
    padding-bottom: 13px; flex-shrink: 0;
  }
  .slot-preview {
    background: rgba(201,168,76,0.08);
    border: 1px solid rgba(201,168,76,0.25);
    border-radius: 10px; padding: 10px 16px;
    font-size: 13px; color: var(--gold);
    font-weight: 600; text-align: center;
    display: flex; align-items: center; justify-content: center; gap: 8px;
  }
  .sms-note {
    background: rgba(34,197,94,0.06);
    border: 1px solid rgba(34,197,94,0.2);
    border-radius: 10px; padding: 10px 16px;
    font-size: 12px; color: #4ade80;
    text-align: center;
    display: flex; align-items: center; justify-content: center; gap: 8px;
  }

  /* Buttons */
  .btn-gold {
    padding: 14px 24px; border-radius: 12px; border: none;
    background: linear-gradient(135deg, var(--gold), var(--gold2), var(--gold));
    background-size: 200% auto;
    color: #0a0e1a; font-family: 'DM Sans', sans-serif;
    font-size: 15px; font-weight: 700;
    cursor: pointer; width: 100%;
    transition: all 0.3s;
    display: flex; align-items: center; justify-content: center; gap: 10px;
    animation: shimmer 3s linear infinite;
    letter-spacing: 0.5px;
  }
  .btn-gold:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(201,168,76,0.4);
    animation: pulse-gold 1.5s ease infinite;
  }
  .btn-gold:disabled { opacity: 0.6; cursor: not-allowed; transform: none; }

  .btn-blue {
    padding: 8px 14px; border-radius: 9px; border: none;
    background: rgba(59,130,246,0.15);
    border: 1px solid rgba(59,130,246,0.3);
    color: #93c5fd; font-family: 'DM Sans', sans-serif;
    font-size: 12px; font-weight: 600;
    cursor: pointer; transition: all 0.2s;
    display: flex; align-items: center; gap: 6px;
  }
  .btn-blue:hover { background: rgba(59,130,246,0.25); color: #bfdbfe; transform: translateY(-1px); }

  .btn-red {
    padding: 8px 14px; border-radius: 9px; border: none;
    background: rgba(239,68,68,0.12);
    border: 1px solid rgba(239,68,68,0.25);
    color: #fca5a5; font-family: 'DM Sans', sans-serif;
    font-size: 12px; font-weight: 600;
    cursor: pointer; transition: all 0.2s;
    display: flex; align-items: center; gap: 6px;
  }
  .btn-red:hover { background: rgba(239,68,68,0.22); transform: translateY(-1px); }

  .btn-green {
    padding: 8px 14px; border-radius: 9px; border: none;
    background: rgba(34,197,94,0.12);
    border: 1px solid rgba(34,197,94,0.25);
    color: #86efac; font-family: 'DM Sans', sans-serif;
    font-size: 12px; font-weight: 600;
    cursor: pointer; transition: all 0.2s;
    display: flex; align-items: center; gap: 6px;
  }
  .btn-green:hover { background: rgba(34,197,94,0.22); transform: translateY(-1px); }

  .btn-outline {
    padding: 8px 14px; border-radius: 9px;
    border: 1px solid rgba(255,255,255,0.1);
    background: transparent; color: #9ca3af;
    font-family: 'DM Sans', sans-serif;
    font-size: 12px; font-weight: 600;
    cursor: pointer; transition: all 0.2s;
    display: flex; align-items: center; gap: 6px;
  }
  .btn-outline:hover { border-color: var(--gold); color: var(--gold); }

  .btn-admin {
    padding: 11px 20px; border-radius: 10px; border: none;
    background: linear-gradient(135deg, var(--gold), var(--gold2));
    color: #0a0e1a; font-family: 'DM Sans', sans-serif;
    font-size: 13px; font-weight: 700;
    cursor: pointer; transition: all 0.25s;
    white-space: nowrap;
    display: flex; align-items: center; gap: 7px;
  }
  .btn-admin:hover { transform: translateY(-2px); box-shadow: 0 6px 20px rgba(201,168,76,0.35); }

  /* Badge */
  .badge-gold {
    background: rgba(201,168,76,0.12);
    border: 1px solid rgba(201,168,76,0.25);
    color: var(--gold); padding: 4px 12px;
    border-radius: 20px; font-size: 11px;
    font-weight: 700; letter-spacing: 0.5px;
  }

  /* Booking cards */
  .booking-item {
    padding: 16px 20px; border-radius: 14px;
    background: rgba(255,255,255,0.03);
    border: 1px solid rgba(201,168,76,0.1);
    transition: all 0.25s;
    display: flex; justify-content: space-between;
    align-items: center; gap: 12px; flex-wrap: wrap;
  }
  .booking-item:hover {
    border-color: rgba(201,168,76,0.3);
    background: rgba(201,168,76,0.04);
    transform: translateX(4px);
  }
  .booking-name {
    font-family: 'Playfair Display', serif;
    font-size: 16px; font-weight: 700; color: #fff;
  }
  .booking-meta { font-size: 13px; color: var(--gold); margin-top: 3px; }
  .booking-phone { font-size: 12px; color: var(--muted); margin-top: 2px; display: flex; align-items: center; gap: 5px; }

  /* Admin grid */
  .admin-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 16px; margin-top: 16px;
  }
  .admin-card {
    padding: 18px; border-radius: 16px;
    background: rgba(255,255,255,0.04);
    border: 1px solid rgba(201,168,76,0.12);
    transition: all 0.25s;
  }
  .admin-card:hover { border-color: rgba(201,168,76,0.3); transform: translateY(-2px); box-shadow: 0 8px 24px rgba(0,0,0,0.3); }
  .admin-name {
    font-family: 'Playfair Display', serif;
    font-size: 16px; font-weight: 700; color: #fff; margin-bottom: 8px;
  }
  .admin-meta { font-size: 12px; color: #94a3b8; margin-top: 4px; display: flex; align-items: center; gap: 6px; }
  .admin-btns { display: flex; gap: 7px; margin-top: 12px; flex-wrap: wrap; }

  /* Toast */
  .toast {
    position: fixed; top: 76px; left: 50%; transform: translateX(-50%);
    padding: 12px 24px; border-radius: 50px;
    font-weight: 700; font-size: 13px; color: #fff;
    z-index: 9999; white-space: nowrap;
    max-width: 90vw; text-align: center;
    display: flex; align-items: center; gap: 10px;
    box-shadow: 0 8px 32px rgba(0,0,0,0.5);
    animation: bounce-in 0.4s ease;
  }
  .toast-success { background: linear-gradient(135deg, #065f46, #059669); border: 1px solid #10b981; }
  .toast-error   { background: linear-gradient(135deg, #7f1d1d, #dc2626); border: 1px solid #ef4444; }

  /* Section */
  .section-head {
    display: flex; justify-content: space-between;
    align-items: center; margin-bottom: 16px;
  }
  .section-title {
    font-family: 'Playfair Display', serif;
    font-size: 20px; font-weight: 700; color: #fff;
    display: flex; align-items: center; gap: 10px;
  }

  /* Empty state */
  .empty {
    text-align: center; padding: 48px 0;
    color: var(--muted); font-size: 14px;
  }
  .empty i { font-size: 36px; margin-bottom: 12px; color: rgba(201,168,76,0.2); display: block; }

  /* Error */
  .err-text { color: #f87171; font-size: 13px; margin-top: 10px; display: flex; align-items: center; gap: 6px; }

  /* Footer */
  .footer {
    text-align: center; padding: 40px 20px;
    border-top: 1px solid rgba(201,168,76,0.1);
    position: relative; z-index: 1;
    margin-top: 40px;
  }
  .footer-brand {
    font-family: 'Playfair Display', serif;
    font-size: 18px; font-weight: 900;
    background: linear-gradient(135deg, var(--gold), var(--gold2));
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    margin-bottom: 12px;
  }
  .footer-social { display: flex; justify-content: center; gap: 14px; margin: 16px 0; }
  .social-btn {
    width: 40px; height: 40px; border-radius: 50%;
    border: 1px solid rgba(201,168,76,0.2);
    background: rgba(201,168,76,0.06);
    color: var(--gold); font-size: 16px;
    display: flex; align-items: center; justify-content: center;
    cursor: pointer; transition: all 0.25s; text-decoration: none;
  }
  .social-btn:hover {
    background: rgba(201,168,76,0.15);
    border-color: var(--gold);
    transform: translateY(-3px);
    box-shadow: 0 6px 20px rgba(201,168,76,0.25);
  }
  .footer-copy { font-size: 12px; color: var(--muted); }

  /* Divider */
  .gold-divider {
    height: 1px; width: 100%;
    background: linear-gradient(90deg, transparent, rgba(201,168,76,0.3), transparent);
    margin: 4px 0;
  }

  /* Scroll bar */
  ::-webkit-scrollbar { width: 5px; }
  ::-webkit-scrollbar-track { background: var(--dark); }
  ::-webkit-scrollbar-thumb { background: rgba(201,168,76,0.3); border-radius: 3px; }

  /* Responsive */
  @media (max-width: 600px) {
    .time-row { flex-direction: column; }
    .time-sep { display: none; }
    .admin-grid { grid-template-columns: 1fr; }
    .hero-stats { gap: 24px; }
  }
`;

/* ── Inject CSS ── */
if (!document.getElementById("premium-css")) {
  const style = document.createElement("style");
  style.id = "premium-css";
  style.textContent = globalCSS;
  document.head.appendChild(style);
}

/* ── Toast ── */
function Toast({ msg }) {
  if (!msg.text) return null;
  return (
    <div className={`toast ${msg.type === "error" ? "toast-error" : "toast-success"}`}>
      <i className={`fa-solid ${msg.type === "error" ? "fa-circle-xmark" : "fa-circle-check"}`}></i>
      {msg.text}
    </div>
  );
}

/* ── MAIN APP ── */
export default function App() {
  const [page, setPage]             = useState("home");
  const [toast, setToast]           = useState({ text:"", type:"" });
  const [bookings, setBookings]     = useState([]);
  const [loadingB, setLoadingB]     = useState(false);
  const [form, setForm]             = useState({ name:"", mobile:"", date:today(), startTime:"", endTime:"" });
  const [submitting, setSubmitting] = useState(false);
  const [adminKey, setAdminKey]     = useState(() => localStorage.getItem("bca_admin_key") || "");
  const [adminBookings, setAdminBookings] = useState([]);
  const [adminLoading, setAdminLoading]   = useState(false);
  const [adminError, setAdminError]       = useState("");
  const [editId, setEditId]               = useState(null);
  const [editForm, setEditForm]           = useState({});
  const [smsDate, setSmsDate]             = useState(today());
  const [smsLoading, setSmsLoading]       = useState(false);

  const flash = (text, type = "success") => {
    setToast({ text, type });
    setTimeout(() => setToast({ text:"", type:"" }), 3500);
  };

  const loadBookings = useCallback(async () => {
    setLoadingB(true);
    try {
      const r = await fetch(`${API}/bookings`);
      if (!r.ok) throw new Error("Server error");
      const data = await r.json();
      if (!Array.isArray(data)) throw new Error("Bad response");
      setBookings(data);
    } catch (e) { flash(e.message, "error"); }
    finally { setLoadingB(false); }
  }, []);

  const loadAdmin = useCallback(async (key) => {
    const k = key !== undefined ? key : adminKey;
    if (!k) return;
    setAdminLoading(true); setAdminError("");
    try {
      const r = await fetch(`${API}/admin/bookings`, { headers: { "x-admin-key": k } });
      const data = await r.json();
      if (!r.ok) throw new Error(data.error || "Unauthorized");
      setAdminBookings(data);
      localStorage.setItem("bca_admin_key", k);
    } catch (e) { setAdminError(e.message); setAdminBookings([]); }
    finally { setAdminLoading(false); }
  }, [adminKey]);

  useEffect(() => { loadBookings(); }, [loadBookings]);
  useEffect(() => { if (page === "admin" && adminKey) loadAdmin(adminKey); }, [page, adminKey, loadAdmin]);

  const handleBook = async (e) => {
    e.preventDefault();
    const { name, mobile, date, startTime, endTime } = form;
    if (!name || !mobile || !date || !startTime || !endTime) return flash("All fields are required", "error");
    if (!/^[6-9]\d{9}$/.test(mobile)) return flash("Enter valid 10-digit mobile", "error");
    if (startTime >= endTime) return flash("End time must be after start time", "error");
    setSubmitting(true);
    try {
      const r = await fetch(`${API}/bookings`, {
        method:"POST", headers:{"Content-Type":"application/json"}, body:JSON.stringify(form),
      });
      const data = await r.json();
      if (!r.ok) throw new Error(data.error);
      flash("Booking confirmed! SMS sent to your mobile.");
      setForm({ name:"", mobile:"", date:today(), startTime:"", endTime:"" });
      loadBookings();
    } catch (e) { flash(e.message, "error"); }
    finally { setSubmitting(false); }
  };

  const handleDelete = async (id) => {
    if (!window.confirm("Delete this booking?")) return;
    try {
      const r = await fetch(`${API}/admin/bookings/${id}`, { method:"DELETE", headers:{"x-admin-key":adminKey} });
      if (!r.ok) throw new Error("Delete failed");
      flash("Booking deleted"); loadAdmin(adminKey);
    } catch (e) { flash(e.message, "error"); }
  };

  const handleUpdate = async (id) => {
    try {
      const r = await fetch(`${API}/admin/bookings/${id}`, {
        method:"PUT", headers:{"Content-Type":"application/json","x-admin-key":adminKey}, body:JSON.stringify(editForm),
      });
      if (!r.ok) throw new Error("Update failed");
      flash("Booking updated"); setEditId(null); loadAdmin(adminKey);
    } catch (e) { flash(e.message, "error"); }
  };

  const handleSingleSMS = async (b) => {
    try {
      const r = await fetch(`${API}/admin/sms`, {
        method:"POST", headers:{"Content-Type":"application/json","x-admin-key":adminKey},
        body:JSON.stringify({ mobile:b.mobile, name:b.name, date:b.date, startTime:b.startTime, endTime:b.endTime }),
      });
      const data = await r.json();
      if (!r.ok) throw new Error(data.error);
      flash("SMS sent to " + b.name);
    } catch (e) { flash(e.message, "error"); }
  };

  const handleSmsAll = async () => {
    if (!smsDate) return flash("Select a date first", "error");
    setSmsLoading(true);
    try {
      const r = await fetch(`${API}/admin/sms-all`, {
        method:"POST", headers:{"Content-Type":"application/json","x-admin-key":adminKey}, body:JSON.stringify({ date:smsDate }),
      });
      const data = await r.json();
      if (!r.ok) throw new Error(data.error);
      flash(data.message);
    } catch (e) { flash(e.message, "error"); }
    finally { setSmsLoading(false); }
  };

  const chartData = (() => {
    const map = {};
    adminBookings.forEach(b => { map[b.date] = (map[b.date] || 0) + 1; });
    return Object.entries(map).sort(([a],[b])=>a.localeCompare(b)).map(([date,count])=>({date,count}));
  })();

  const endTimeOptions = TIME_OPTIONS.filter(t => t.value > form.startTime);
  const editEndOptions = TIME_OPTIONS.filter(t => t.value > (editForm.startTime || ""));
  const fmtTime = (val) => { const f = TIME_OPTIONS.find(t=>t.value===val); return f ? f.label : val||""; };

  return (
    <div style={{ position:"relative", zIndex:1 }}>
      <Toast msg={toast} />

      {/* NAV */}
      <nav className="nav-premium">
        <div className="nav-inner">
          <div className="nav-logo">
            <i className="fa-solid fa-baseball-bat-ball"></i>
            VSR Box Cricket
          </div>
          <div className="nav-links">
            <button className={`nav-btn ${page==="home"?"active":""}`} onClick={()=>setPage("home")}>
              <i className="fa-solid fa-house"></i> Home
            </button>
            <span className="nav-divider">|</span>
            <button className={`nav-btn ${page==="admin"?"active":""}`} onClick={()=>setPage("admin")}>
              <i className="fa-solid fa-shield-halved"></i> Admin
            </button>
          </div>
        </div>
      </nav>

      {/* MAIN */}
      <main style={{ maxWidth:960, margin:"0 auto", padding:"0 16px 60px", position:"relative", zIndex:1 }}>
        {page === "home" ? (
          <HomePage bookings={bookings} loading={loadingB} form={form} setForm={setForm}
            submitting={submitting} handleBook={handleBook} refresh={loadBookings}
            endTimeOptions={endTimeOptions} fmtTime={fmtTime} />
        ) : (
          <AdminPage adminKey={adminKey} setAdminKey={setAdminKey}
            adminBookings={adminBookings} adminLoading={adminLoading}
            adminError={adminError} chartData={chartData}
            editId={editId} editForm={editForm} setEditForm={setEditForm}
            editEndOptions={editEndOptions} smsDate={smsDate} setSmsDate={setSmsDate}
            smsLoading={smsLoading} onLoad={loadAdmin}
            onEdit={(b)=>{setEditId(b._id);setEditForm({name:b.name,mobile:b.mobile,date:b.date,startTime:b.startTime,endTime:b.endTime});}}
            onCancelEdit={()=>setEditId(null)} onUpdate={handleUpdate} onDelete={handleDelete}
            onSingleSMS={handleSingleSMS} onSmsAll={handleSmsAll} fmtTime={fmtTime} />
        )}
      </main>

      {/* FOOTER */}
      <footer className="footer">
        <div className="footer-brand">
          <i className="fa-solid fa-baseball-bat-ball"></i> VSR Box Cricket Academy
        </div>
        <div className="gold-divider" style={{maxWidth:200,margin:"0 auto 16px"}}></div>
        <div className="footer-social">
          <a href="https://instagram.com" target="_blank" rel="noreferrer" className="social-btn" title="Instagram">
            <i className="fa-brands fa-instagram"></i>
          </a>
          <a href="https://facebook.com" target="_blank" rel="noreferrer" className="social-btn" title="Facebook">
            <i className="fa-brands fa-facebook-f"></i>
          </a>
          <a href="https://wa.me/918688496208" target="_blank" rel="noreferrer" className="social-btn" title="WhatsApp">
            <i className="fa-brands fa-whatsapp"></i>
          </a>
          <a href="https://youtube.com" target="_blank" rel="noreferrer" className="social-btn" title="YouTube">
            <i className="fa-brands fa-youtube"></i>
          </a>
          <a href="tel:+918688496208" className="social-btn" title="Call">
            <i className="fa-solid fa-phone"></i>
          </a>
        </div>
        <p className="footer-copy">
          &copy; 2026 VSR Box Cricket Academy · Built with <i className="fa-solid fa-heart" style={{color:"#ef4444"}}></i> by <strong style={{color:"var(--gold)"}}>CodeWithK</strong>
        </p>
      </footer>
    </div>
  );
}

/* ── HOME PAGE ── */
function HomePage({ bookings, loading, form, setForm, submitting, handleBook, refresh, endTimeOptions, fmtTime }) {
  return (
    <div style={{display:"flex",flexDirection:"column",gap:24}}>

      {/* HERO */}
      <div className="hero-section">
        <div className="badge-gold animate-fadeup" style={{marginBottom:16}}>
          <i className="fa-solid fa-star"></i> Premium Booking
        </div>
        <h1 className="hero-title animate-fadeup delay-1">Booked Slots</h1>
        <p className="hero-sub animate-fadeup delay-2">
          Reserve your box cricket slot instantly. Fast, simple, and beautiful.
        </p>
        <div className="hero-stats animate-fadeup delay-3">
          <div className="hero-stat">
            <div className="hero-stat-num">{bookings.length}</div>
            <div className="hero-stat-label"><i className="fa-solid fa-calendar-check"></i> Total Bookings</div>
          </div>
          <div className="hero-stat">
            <div className="hero-stat-num">24/7</div>
            <div className="hero-stat-label"><i className="fa-solid fa-clock"></i> Available</div>
          </div>
          <div className="hero-stat">
            <div className="hero-stat-num">SMS</div>
            <div className="hero-stat-label"><i className="fa-solid fa-mobile-screen"></i> Confirmation</div>
          </div>
        </div>
      </div>

      {/* BOOK FORM */}
      <div className="card-premium animate-fadeup delay-2">
        <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:20,flexWrap:"wrap",gap:12}}>
          <div>
            <div className="card-title">
              <i className="fa-solid fa-calendar-plus" style={{color:"var(--gold)"}}></i>
              Book a Slot
            </div>
            <div className="card-sub">Choose your start and end time freely.</div>
          </div>
          <span className="badge-gold">{bookings.length} bookings</span>
        </div>

        <form onSubmit={handleBook} style={{display:"flex",flexDirection:"column",gap:16}}>
          <div className="form-field">
            <label className="form-label">
              <i className="fa-solid fa-user"></i> Full Name
            </label>
            <input className="form-input" placeholder="Your name" value={form.name}
              onChange={e=>setForm({...form,name:e.target.value})} />
          </div>

          <div className="form-field">
            <label className="form-label">
              <i className="fa-solid fa-mobile-screen-button"></i> Mobile Number
            </label>
            <input className="form-input" placeholder="10-digit mobile" maxLength={10} value={form.mobile}
              onChange={e=>setForm({...form,mobile:e.target.value.replace(/\D/,"")})} />
          </div>

          <div className="form-field">
            <label className="form-label">
              <i className="fa-solid fa-calendar-days"></i> Date
            </label>
            <input type="date" className="form-input" value={form.date} min={today()}
              onChange={e=>setForm({...form,date:e.target.value})} />
          </div>

          <div className="time-row">
            <div className="form-field" style={{flex:1}}>
              <label className="form-label">
                <i className="fa-solid fa-clock"></i> Start Time
              </label>
              <select className="form-input" value={form.startTime}
                onChange={e=>setForm({...form,startTime:e.target.value,endTime:""})}>
                <option value="">-- Start --</option>
                {TIME_OPTIONS.slice(0,-1).map(t=>(
                  <option key={t.value} value={t.value}>{t.label}</option>
                ))}
              </select>
            </div>
            <div className="time-sep"><i className="fa-solid fa-arrow-right"></i></div>
            <div className="form-field" style={{flex:1}}>
              <label className="form-label">
                <i className="fa-solid fa-clock"></i> End Time
              </label>
              <select className="form-input" value={form.endTime}
                onChange={e=>setForm({...form,endTime:e.target.value})}
                disabled={!form.startTime}>
                <option value="">-- End --</option>
                {endTimeOptions.map(t=>(
                  <option key={t.value} value={t.value}>{t.label}</option>
                ))}
              </select>
            </div>
          </div>

          {form.startTime && form.endTime && (
            <div className="slot-preview">
              <i className="fa-solid fa-circle-check"></i>
              {fmtTime(form.startTime)} → {fmtTime(form.endTime)}
            </div>
          )}

          <div className="sms-note">
            <i className="fa-solid fa-comment-sms"></i>
            Confirmation SMS will be sent to your mobile number
          </div>

          <button type="submit" className="btn-gold" disabled={submitting}>
            {submitting
              ? <><i className="fa-solid fa-spinner fa-spin"></i> Booking...</>
              : <><i className="fa-solid fa-rocket"></i> Confirm Booking</>
            }
          </button>
        </form>
      </div>

      {/* RECENT BOOKINGS */}
      <div className="card-premium animate-fadeup delay-3">
        <div className="section-head">
          <div className="section-title">
            <i className="fa-solid fa-users" style={{color:"var(--gold)"}}></i>
            Recent Bookings
          </div>
          <div style={{display:"flex",alignItems:"center",gap:10}}>
            <span className="badge-gold">{bookings.length} total</span>
            <button className="btn-outline" onClick={refresh}>
              <i className="fa-solid fa-rotate-right"></i>
            </button>
          </div>
        </div>
        <div className="gold-divider"></div>
        <div style={{marginTop:16}}>
          {loading ? (
            <div className="empty">
              <i className="fa-solid fa-spinner fa-spin"></i>
              <div style={{marginTop:12}}>Loading bookings...</div>
            </div>
          ) : bookings.length === 0 ? (
            <div className="empty">
              <i className="fa-solid fa-calendar-xmark"></i>
              <div>No bookings yet. Be the first!</div>
            </div>
          ) : (
            <div style={{display:"flex",flexDirection:"column",gap:10}}>
              {bookings.map((b,i)=>(
                <div key={b._id} className="booking-item animate-fadeup" style={{animationDelay:`${i*0.05}s`}}>
                  <div>
                    <div className="booking-name">{b.name}</div>
                    <div className="booking-meta">
                      <i className="fa-solid fa-calendar-day"></i> {b.date} &nbsp;
                      <i className="fa-solid fa-clock"></i> {fmtTime(b.startTime)} – {fmtTime(b.endTime)}
                    </div>
                    <div className="booking-phone">
                      <i className="fa-solid fa-phone"></i> {b.mobile}
                    </div>
                  </div>
                  <i className="fa-solid fa-cricket-bat-ball" style={{color:"rgba(201,168,76,0.3)",fontSize:24}}></i>
                </div>
              ))}
            </div>
          )}
        </div>
      </div>
    </div>
  );
}

/* ── ADMIN PAGE ── */
function AdminPage({
  adminKey, setAdminKey, adminBookings, adminLoading,
  adminError, chartData, editId, editForm, setEditForm,
  editEndOptions, smsDate, setSmsDate, smsLoading,
  onLoad, onEdit, onCancelEdit, onUpdate, onDelete,
  onSingleSMS, onSmsAll, fmtTime,
}) {
  return (
    <div style={{display:"flex",flexDirection:"column",gap:24,paddingTop:32}}>

      {/* Admin Login */}
      <div className="card-premium animate-fadeup">
        <div className="card-title">
          <i className="fa-solid fa-shield-halved" style={{color:"var(--gold)"}}></i>
          Admin Panel
        </div>
        <div className="card-sub">Enter your admin key to manage bookings</div>
        <div style={{display:"flex",gap:10,flexWrap:"wrap"}}>
          <input type="password" className="form-input" style={{flex:1,minWidth:180}}
            placeholder="Paste admin key" value={adminKey}
            onChange={e=>setAdminKey(e.target.value)}
            onKeyDown={e=>e.key==="Enter"&&onLoad(adminKey)} />
          <button className="btn-admin" onClick={()=>onLoad(adminKey)}>
            <i className="fa-solid fa-key"></i> Save & Load
          </button>
        </div>
        {adminError && (
          <div className="err-text">
            <i className="fa-solid fa-triangle-exclamation"></i> {adminError}
          </div>
        )}
      </div>

      {/* Bulk SMS */}
      {adminBookings.length > 0 && (
        <div className="card-premium animate-fadeup delay-1">
          <div className="card-title">
            <i className="fa-solid fa-comment-sms" style={{color:"var(--gold)"}}></i>
            Bulk SMS Reminder
          </div>
          <div className="card-sub">Send reminder SMS to all customers on a specific date</div>
          <div style={{display:"flex",gap:10,flexWrap:"wrap",alignItems:"center"}}>
            <input type="date" className="form-input" style={{flex:1,minWidth:160}}
              value={smsDate} onChange={e=>setSmsDate(e.target.value)} />
            <button className="btn-green" style={{padding:"13px 20px",fontSize:13,fontWeight:700}} onClick={onSmsAll} disabled={smsLoading}>
              {smsLoading
                ? <><i className="fa-solid fa-spinner fa-spin"></i> Sending...</>
                : <><i className="fa-solid fa-paper-plane"></i> Send Reminders</>
              }
            </button>
          </div>
        </div>
      )}

      {/* Chart */}
      {chartData.length > 0 && (
        <div className="card-premium animate-fadeup delay-2">
          <div className="card-title">
            <i className="fa-solid fa-chart-column" style={{color:"var(--gold)"}}></i>
            Bookings per Day
          </div>
          <div style={{marginTop:20}}>
            <ResponsiveContainer width="100%" height={220}>
              <BarChart data={chartData}>
                <CartesianGrid strokeDasharray="3 3" stroke="rgba(201,168,76,0.08)" />
                <XAxis dataKey="date" tick={{fill:"#6b7280",fontSize:11,fontFamily:"DM Sans"}} />
                <YAxis allowDecimals={false} tick={{fill:"#6b7280",fontSize:11}} />
                <Tooltip contentStyle={{background:"#0d1525",border:"1px solid rgba(201,168,76,0.3)",borderRadius:12,color:"#e8dcc8",fontFamily:"DM Sans"}} />
                <Legend wrapperStyle={{color:"#6b7280",fontSize:12,fontFamily:"DM Sans"}} />
                <Bar dataKey="count" name="Bookings" fill="url(#goldGrad)" radius={[6,6,0,0]} />
                <defs>
                  <linearGradient id="goldGrad" x1="0" y1="0" x2="0" y2="1">
                    <stop offset="0%" stopColor="#f0d080" />
                    <stop offset="100%" stopColor="#c9a84c" />
                  </linearGradient>
                </defs>
              </BarChart>
            </ResponsiveContainer>
          </div>
        </div>
      )}

      {/* Bookings */}
      <div className="card-premium animate-fadeup delay-3">
        <div className="section-head">
          <div className="section-title">
            <i className="fa-solid fa-sliders" style={{color:"var(--gold)"}}></i>
            Controls
          </div>
          <span className="badge-gold">{adminBookings.length} bookings</span>
        </div>
        <div className="gold-divider"></div>

        {adminLoading ? (
          <div className="empty">
            <i className="fa-solid fa-spinner fa-spin"></i>
            <div style={{marginTop:12}}>Loading admin data...</div>
          </div>
        ) : adminBookings.length === 0 ? (
          <div className="empty">
            <i className="fa-solid fa-folder-open"></i>
            <div>{adminError ? "Authorization failed. Enter correct admin key." : "No bookings found."}</div>
          </div>
        ) : (
          <div className="admin-grid">
            {adminBookings.map(b=>(
              <div key={b._id} className="admin-card">
                {editId === b._id ? (
                  <div style={{display:"flex",flexDirection:"column",gap:10}}>
                    <input className="form-input" value={editForm.name} placeholder="Name"
                      onChange={e=>setEditForm({...editForm,name:e.target.value})} />
                    <input className="form-input" value={editForm.mobile} placeholder="Mobile"
                      onChange={e=>setEditForm({...editForm,mobile:e.target.value})} />
                    <input type="date" className="form-input" value={editForm.date}
                      onChange={e=>setEditForm({...editForm,date:e.target.value})} />
                    <select className="form-input" value={editForm.startTime}
                      onChange={e=>setEditForm({...editForm,startTime:e.target.value,endTime:""})}>
                      <option value="">-- Start --</option>
                      {TIME_OPTIONS.slice(0,-1).map(t=><option key={t.value} value={t.value}>{t.label}</option>)}
                    </select>
                    <select className="form-input" value={editForm.endTime}
                      onChange={e=>setEditForm({...editForm,endTime:e.target.value})}>
                      <option value="">-- End --</option>
                      {editEndOptions.map(t=><option key={t.value} value={t.value}>{t.label}</option>)}
                    </select>
                    <div style={{display:"flex",gap:8}}>
                      <button className="btn-green" style={{flex:1,padding:"10px"}} onClick={()=>onUpdate(b._id)}>
                        <i className="fa-solid fa-floppy-disk"></i> Save
                      </button>
                      <button className="btn-outline" onClick={onCancelEdit}>
                        <i className="fa-solid fa-xmark"></i> Cancel
                      </button>
                    </div>
                  </div>
                ) : (
                  <>
                    <div className="admin-name">{b.name}</div>
                    <div className="admin-meta"><i className="fa-solid fa-calendar-day"></i> {b.date}</div>
                    <div className="admin-meta"><i className="fa-solid fa-clock"></i> {fmtTime(b.startTime)} – {fmtTime(b.endTime)}</div>
                    <div className="admin-meta"><i className="fa-solid fa-phone"></i> {b.mobile}</div>
                    <div className="admin-btns">
                      <button className="btn-blue" onClick={()=>onEdit(b)}>
                        <i className="fa-solid fa-pen"></i> Edit
                      </button>
                      <button className="btn-red" onClick={()=>onDelete(b._id)}>
                        <i className="fa-solid fa-trash"></i> Delete
                      </button>
                      <button className="btn-green" onClick={()=>onSingleSMS(b)}>
                        <i className="fa-solid fa-comment-sms"></i> SMS
                      </button>
                    </div>
                  </>
                )}
              </div>
            ))}
          </div>
        )}
      </div>
    </div>
  );
}
