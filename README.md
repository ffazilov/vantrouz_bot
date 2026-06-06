import { useState, useRef } from "react";

// ── ADMIN CREDENTIALS ──────────────────────────────────────
const ADMIN_USERNAME = "fazilov";
const ADMIN_PASSWORD = "Nkjmvhgb1.";
const SUPPORT_TG = "vantro_admin";

// ── INITIAL DATA ───────────────────────────────────────────
const INITIAL_PRODUCTS = [
  {
    id: 1, name: "Qora charm sumka", price: 250000,
    description: "Italiya charm sumkasi, premium sifat. Katta sig'im, davomli material.",
    image: "https://images.unsplash.com/photo-1548036328-c9fa89d128fa?w=500&q=80",
    sizes: ["S", "M", "L"], category: "Sumkalar", gender: "ayol",
    reviews: [{ author: "Malika", phone: "+998901112233", text: "Zo'r mahsulot!", rating: 5 }],
  },
  {
    id: 2, name: "Oltin bracelet", price: 180000,
    description: "18k oltin qoplama, nafis dizayn. Har qanday kiyim bilan mos.",
    image: "https://images.unsplash.com/photo-1611591437281-460bfbe1220a?w=500&q=80",
    sizes: ["XS", "S", "M"], category: "Aksesuar", gender: "ayol",
    reviews: [],
  },
  {
    id: 3, name: "Elegance ko'ylak", price: 320000,
    description: "Ipak aralash material, rasmiy uslub. Bayram va tadbirlarga ideal.",
    image: "https://images.unsplash.com/photo-1595777457583-95e059d581b8?w=500&q=80",
    sizes: ["XS", "S", "M", "L", "XL"], category: "Kiyimlar", gender: "ayol",
    reviews: [{ author: "Zulfiya", phone: "+998991234567", text: "Judayam chiroyli!", rating: 5 }],
  },
  {
    id: 4, name: "Erkaklar ko'ylagi", price: 290000,
    description: "Klassik uslub, premium paxta material. Ish va sayr uchun.",
    image: "https://images.unsplash.com/photo-1602810318383-e386cc2a3ccf?w=500&q=80",
    sizes: ["S", "M", "L", "XL", "XXL"], category: "Kiyimlar", gender: "erkak",
    reviews: [],
  },
  {
    id: 5, name: "Sport soat", price: 420000,
    description: "Suv o'tkazmaydigan, sport va kundalik foydalanish uchun.",
    image: "https://images.unsplash.com/photo-1523275335684-37898b6baf30?w=500&q=80",
    sizes: ["Standart"], category: "Aksesuar", gender: "erkak",
    reviews: [],
  },
];

const CATS = ["Hammasi", "Kiyimlar", "Sumkalar", "Aksesuar"];
const GENDERS = ["Barchasi", "Ayollar", "Erkaklar"];
const formatPrice = (n) => Number(n).toLocaleString("uz-UZ") + " so'm";

// ── STYLES ─────────────────────────────────────────────────
const S = {
  gold: "#D4AF37", silver: "#C0C0C0", bg: "#000000", card: "#0d0d0d", border: "#1e1e1e",
  inp: { width: "100%", padding: "13px 16px", background: "#0d0d0d", border: "1px solid #2a2a2a", borderRadius: 6, color: "#C0C0C0", fontFamily: "'Montserrat',sans-serif", fontSize: 14, outline: "none", boxSizing: "border-box" },
  btn: (full) => ({ width: full ? "100%" : "auto", padding: "14px 20px", background: "linear-gradient(135deg,#D4AF37,#b8960c)", color: "#000", border: "none", borderRadius: 6, fontWeight: 700, fontSize: 13, letterSpacing: 1.5, fontFamily: "'Montserrat',sans-serif", cursor: "pointer" }),
  ghost: { padding: "12px 20px", background: "transparent", color: "#C0C0C0", border: "1px solid #2a2a2a", borderRadius: 6, fontFamily: "'Montserrat',sans-serif", fontSize: 13, cursor: "pointer" },
  label: { display: "block", color: "#D4AF37", fontSize: 11, letterSpacing: 2, marginBottom: 8, fontFamily: "'Montserrat',sans-serif" },
};

export default function App() {
  // ── STATE ──
  const [screen, setScreen] = useState("auth");
  const [user, setUser] = useState(null); // { phone, nickname, addresses:[], myOrders:[] }
  const [products, setProducts] = useState(INITIAL_PRODUCTS);
  const [cart, setCart] = useState([]);
  const [selProd, setSelProd] = useState(null);
  const [selSize, setSelSize] = useState("");
  const [activeCat, setActiveCat] = useState("Hammasi");
  const [activeGender, setActiveGender] = useState("Barchasi");
  const [orders, setOrders] = useState([]);
  const [notif, setNotif] = useState("");
  const [reviewText, setReviewText] = useState("");
  const [reviewRating, setReviewRating] = useState(5);
  const [reviewProd, setReviewProd] = useState(null);

  // Auth state
  const [authPhone, setAuthPhone] = useState("");
  const [authNick, setAuthNick] = useState("");
  const [authStep, setAuthStep] = useState("phone"); // phone | nick

  // Profile edit
  const [editNick, setEditNick] = useState("");
  const [editPhone, setEditPhone] = useState("");
  const [newAddr, setNewAddr] = useState("");
  const [profileEdit, setProfileEdit] = useState(""); // "nick"|"phone"|"addr"|""

  // Cart / order
  const [selAddr, setSelAddr] = useState("");
  const [manualAddr, setManualAddr] = useState("");

  // Admin
  const [adminScreen, setAdminScreen] = useState("login"); // login|panel
  const [adminUser, setAdminUser] = useState("");
  const [adminPass, setAdminPass] = useState("");
  const [adminTab, setAdminTab] = useState("orders");
  const [newProd, setNewProd] = useState({ name: "", price: "", description: "", image: "", category: "Kiyimlar", gender: "ayol", sizes: "" });
  const [editProd, setEditProd] = useState(null);
  const [imgFiles, setImgFiles] = useState({});
  const fileRef = useRef();
  const editFileRef = useRef();

  const showNotif = (msg, dur = 3000) => { setNotif(msg); setTimeout(() => setNotif(""), dur); };

  // ── AUTH ──
  const handlePhone = () => {
    const c = authPhone.replace(/\D/g, "");
    if (c.length < 9) return showNotif("Raqamni to'g'ri kiriting");
    setAuthStep("nick");
  };
  const handleNick = () => {
    if (!authNick.trim()) return showNotif("Nik kiriting");
    setUser({ phone: "+998" + authPhone.replace(/\D/g, "").slice(-9), nickname: authNick.trim(), addresses: [], myOrders: [] });
    setScreen("main");
  };

  // ── CART ──
  const addToCart = (prod, size) => {
    if (!size) return showNotif("Razmer tanlang!");
    const ex = cart.find(i => i.id === prod.id && i.size === size);
    if (ex) setCart(cart.map(i => i.id === prod.id && i.size === size ? { ...i, qty: i.qty + 1 } : i));
    else setCart([...cart, { ...prod, size, qty: 1 }]);
    showNotif("✓ Savatga qo'shildi");
    setScreen("main");
  };
  const removeCart = (id, size) => setCart(cart.filter(i => !(i.id === id && i.size === size)));
  const totalCart = cart.reduce((s, i) => s + i.price * i.qty, 0);
  const totalQty = cart.reduce((s, i) => s + i.qty, 0);

  // ── ORDER ──
  const placeOrder = () => {
    const addr = selAddr === "__manual__" ? manualAddr : selAddr;
    if (!addr) return showNotif("Manzil tanlang yoki kiriting!");
    if (!cart.length) return showNotif("Savat bo'sh!");
    const order = {
      id: Date.now(), phone: user.phone, nickname: user.nickname, address: addr,
      items: [...cart], total: totalCart, date: new Date().toLocaleString("uz"),
    };
    setOrders(prev => [order, ...prev]);
    setUser(u => ({ ...u, myOrders: [order, ...(u.myOrders || [])] }));
    setCart([]);
    setSelAddr("");
    setManualAddr("");
    showNotif("✅ Buyurtmangiz qabul qilindi!");
    setScreen("main");
  };

  // ── REVIEW ──
  const submitReview = () => {
    if (!reviewText) return showNotif("Fikr yozing");
    setProducts(ps => ps.map(p => p.id === reviewProd.id
      ? { ...p, reviews: [...p.reviews, { author: user.nickname, phone: user.phone, text: reviewText, rating: reviewRating }] }
      : p));
    setReviewText(""); setReviewRating(5);
    showNotif("✓ Fikringiz qo'shildi");
    setScreen("product");
  };

  // ── PROFILE ──
  const saveNick = () => { if (!editNick.trim()) return showNotif("Nik bo'sh bo'lmasin"); setUser(u => ({ ...u, nickname: editNick.trim() })); setProfileEdit(""); showNotif("✓ Saqlandi"); };
  const savePhone = () => { if (editPhone.replace(/\D/g, "").length < 9) return showNotif("Raqam noto'g'ri"); setUser(u => ({ ...u, phone: "+998" + editPhone.replace(/\D/g, "").slice(-9) })); setProfileEdit(""); showNotif("✓ Saqlandi"); };
  const addAddress = () => { if (!newAddr.trim()) return showNotif("Manzil kiriting"); setUser(u => ({ ...u, addresses: [...u.addresses, newAddr.trim()] })); setNewAddr(""); setProfileEdit(""); showNotif("✓ Manzil qo'shildi"); };
  const removeAddress = (i) => setUser(u => ({ ...u, addresses: u.addresses.filter((_, idx) => idx !== i) }));

  // ── ADMIN ──
  const adminLogin = () => {
    if (adminUser === ADMIN_USERNAME && adminPass === ADMIN_PASSWORD) { setAdminScreen("panel"); }
    else showNotif("Noto'g'ri login yoki parol");
  };

  const handleAddProd = () => {
    if (!newProd.name || !newProd.price) return showNotif("Nom va narx kerak");
    const imgKey = "new_" + Date.now();
    const imgSrc = imgFiles[imgKey] || newProd.image || "https://images.unsplash.com/photo-1441984904996-e0b6ba687e04?w=500";
    setProducts(ps => [...ps, { ...newProd, id: Date.now(), price: parseInt(newProd.price), sizes: newProd.sizes ? newProd.sizes.split(",").map(s => s.trim()).filter(Boolean) : ["S", "M", "L"], reviews: [], image: imgSrc }]);
    setNewProd({ name: "", price: "", description: "", image: "", category: "Kiyimlar", gender: "ayol", sizes: "" });
    showNotif("✅ Mahsulot qo'shildi");
  };

  const handleDeleteProd = (id) => { setProducts(ps => ps.filter(p => p.id !== id)); showNotif("🗑 O'chirildi"); };

  const handleEditSave = () => {
    if (!editProd.name || !editProd.price) return showNotif("Nom va narx kerak");
    const imgSrc = imgFiles["edit_" + editProd.id] || editProd.image;
    setProducts(ps => ps.map(p => p.id === editProd.id ? { ...editProd, price: parseInt(editProd.price), sizes: typeof editProd.sizes === "string" ? editProd.sizes.split(",").map(s => s.trim()).filter(Boolean) : editProd.sizes, image: imgSrc } : p));
    setEditProd(null); showNotif("✅ Saqlandi");
  };

  const handleFileUpload = (e, key) => {
    const file = e.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (ev) => setImgFiles(prev => ({ ...prev, [key]: ev.target.result }));
    reader.readAsDataURL(file);
  };

  // ── FILTER ──
  const filtered = products.filter(p => {
    const catOk = activeCat === "Hammasi" || p.category === activeCat;
    const genOk = activeGender === "Barchasi" || (activeGender === "Ayollar" ? p.gender === "ayol" : p.gender === "erkak");
    return catOk && genOk;
  });

  const avgRating = (reviews) => reviews.length ? (reviews.reduce((s, r) => s + r.rating, 0) / reviews.length).toFixed(1) : null;

  // ─────────────────── RENDER ────────────────────────────
  return (
    <div style={{ minHeight: "100vh", background: "#000", color: "#C0C0C0", fontFamily: "'Playfair Display',Georgia,serif", maxWidth: 480, margin: "0 auto", paddingBottom: 72, position: "relative" }}>
      <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,600;0,700;1,400&family=Montserrat:wght@300;400;500;600&display=swap" rel="stylesheet" />

      {/* NOTIFICATION */}
      {notif && (
        <div style={{ position: "fixed", top: 20, left: "50%", transform: "translateX(-50%)", background: "#D4AF37", color: "#000", padding: "12px 28px", borderRadius: 8, zIndex: 9999, fontFamily: "'Montserrat',sans-serif", fontWeight: 600, fontSize: 13, whiteSpace: "nowrap", boxShadow: "0 4px 24px rgba(212,175,55,0.5)", letterSpacing: 0.5 }}>
          {notif}
        </div>
      )}

      {/* ══════════════ AUTH ══════════════ */}
      {screen === "auth" && (
        <div style={{ display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", minHeight: "100vh", padding: 32 }}>
          {/* Logo */}
          <div style={{ textAlign: "center", marginBottom: 52 }}>
            <div style={{ width: 72, height: 72, border: "2px solid #D4AF37", borderRadius: "50%", display: "flex", alignItems: "center", justifyContent: "center", margin: "0 auto 16px", fontSize: 30 }}>👑</div>
            <h1 style={{ color: "#D4AF37", fontSize: 34, margin: 0, fontWeight: 700, letterSpacing: 4 }}>LUXE</h1>
            <p style={{ color: "#555", fontSize: 11, fontFamily: "'Montserrat',sans-serif", letterSpacing: 4, margin: "6px 0 0" }}>FASHION & ACCESSORIES</p>
          </div>

          {authStep === "phone" ? (
            <div style={{ width: "100%", maxWidth: 340 }}>
              <label style={S.label}>TELEFON RAQAM</label>
              <div style={{ display: "flex", border: "1px solid #D4AF37", borderRadius: 6, overflow: "hidden", marginBottom: 20 }}>
                <span style={{ padding: "13px 14px", color: "#D4AF37", fontFamily: "'Montserrat',sans-serif", background: "#0d0d0d", borderRight: "1px solid #333", fontSize: 14 }}>+998</span>
                <input type="tel" placeholder="90 123 45 67" value={authPhone} onChange={e => setAuthPhone(e.target.value)} onKeyDown={e => e.key === "Enter" && handlePhone()}
                  style={{ flex: 1, background: "#0d0d0d", border: "none", outline: "none", color: "#C0C0C0", padding: "13px 16px", fontSize: 16, fontFamily: "'Montserrat',sans-serif" }} />
              </div>
              <button onClick={handlePhone} style={{ ...S.btn(true) }}>DAVOM ETISH →</button>
              <div style={{ textAlign: "center", marginTop: 16 }}>
                <button onClick={() => setScreen("adminEntry")} style={{ background: "none", border: "none", color: "#333", cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 12 }}>Admin kirish</button>
              </div>
            </div>
          ) : (
            <div style={{ width: "100%", maxWidth: 340 }}>
              <div style={{ textAlign: "center", marginBottom: 24 }}>
                <p style={{ color: "#555", fontFamily: "'Montserrat',sans-serif", fontSize: 13 }}>Salom! Nikingizni tanlang</p>
              </div>
              <label style={S.label}>SIZNING NIKINGIZ</label>
              <input placeholder="Masalan: Malika_tashkent" value={authNick} onChange={e => setAuthNick(e.target.value)} onKeyDown={e => e.key === "Enter" && handleNick()}
                style={{ ...S.inp, marginBottom: 20, border: "1px solid #D4AF37" }} />
              <button onClick={handleNick} style={S.btn(true)}>KIRISH 👑</button>
              <button onClick={() => setAuthStep("phone")} style={{ ...S.ghost, width: "100%", marginTop: 10 }}>← Orqaga</button>
            </div>
          )}
        </div>
      )}

      {/* ══════════════ ADMIN ENTRY ══════════════ */}
      {screen === "adminEntry" && (
        <div style={{ display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", minHeight: "100vh", padding: 32 }}>
          <div style={{ textAlign: "center", marginBottom: 40 }}>
            <div style={{ fontSize: 40, marginBottom: 12 }}>🛡️</div>
            <h2 style={{ color: "#D4AF37", margin: 0, letterSpacing: 2 }}>ADMIN PANEL</h2>
          </div>
          <div style={{ width: "100%", maxWidth: 320 }}>
            {adminScreen === "login" ? (
              <>
                <label style={S.label}>LOGIN</label>
                <input placeholder="fazilov" value={adminUser} onChange={e => setAdminUser(e.target.value)} style={{ ...S.inp, marginBottom: 14 }} />
                <label style={S.label}>PAROL</label>
                <input type="password" placeholder="••••••••" value={adminPass} onChange={e => setAdminPass(e.target.value)} style={{ ...S.inp, marginBottom: 20 }} />
                <button onClick={adminLogin} style={S.btn(true)}>KIRISH</button>
                <button onClick={() => { setScreen("auth"); setAdminScreen("login"); setAdminUser(""); setAdminPass(""); }} style={{ ...S.ghost, width: "100%", marginTop: 10 }}>← Orqaga</button>
              </>
            ) : (
              <AdminPanel
                products={products} orders={orders} newProd={newProd} setNewProd={setNewProd}
                editProd={editProd} setEditProd={setEditProd} imgFiles={imgFiles}
                handleFileUpload={handleFileUpload} handleAddProd={handleAddProd}
                handleDeleteProd={handleDeleteProd} handleEditSave={handleEditSave}
                adminTab={adminTab} setAdminTab={setAdminTab} formatPrice={formatPrice}
                fileRef={fileRef} editFileRef={editFileRef}
                onLogout={() => { setAdminScreen("login"); setAdminUser(""); setAdminPass(""); setScreen("auth"); }}
                showNotif={showNotif}
              />
            )}
          </div>
        </div>
      )}

      {/* ══════════════ MAIN CATALOG ══════════════ */}
      {screen === "main" && (
        <div>
          {/* Header */}
          <div style={{ position: "sticky", top: 0, background: "#000", zIndex: 50, borderBottom: "1px solid #111" }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "16px 16px 8px" }}>
              <div>
                <h1 style={{ color: "#D4AF37", fontSize: 20, margin: 0, letterSpacing: 3 }}>LUXE</h1>
                <p style={{ color: "#333", fontSize: 10, fontFamily: "'Montserrat',sans-serif", letterSpacing: 2, margin: 0 }}>Salom, {user?.nickname} 👑</p>
              </div>
              <button onClick={() => setScreen("cart")} style={{ background: "none", border: "1px solid #222", borderRadius: 8, color: "#C0C0C0", padding: "8px 16px", cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 12, position: "relative" }}>
                🛒
                {totalQty > 0 && <span style={{ position: "absolute", top: -6, right: -6, background: "#D4AF37", color: "#000", borderRadius: "50%", width: 18, height: 18, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 10, fontWeight: 700 }}>{totalQty}</span>}
              </button>
            </div>

            {/* Gender tabs */}
            <div style={{ display: "flex", padding: "0 16px 8px", gap: 6 }}>
              {GENDERS.map(g => (
                <button key={g} onClick={() => setActiveGender(g)} style={{ flex: 1, padding: "8px 4px", border: activeGender === g ? "none" : "1px solid #222", background: activeGender === g ? "#D4AF37" : "transparent", color: activeGender === g ? "#000" : "#555", borderRadius: 20, cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 11, fontWeight: activeGender === g ? 700 : 400, letterSpacing: 0.5 }}>
                  {g === "Ayollar" ? "♀ Ayollar" : g === "Erkaklar" ? "♂ Erkaklar" : "✦ Barchasi"}
                </button>
              ))}
            </div>

            {/* Category tabs */}
            <div style={{ display: "flex", gap: 6, overflowX: "auto", padding: "0 16px 12px", scrollbarWidth: "none" }}>
              {CATS.map(c => (
                <button key={c} onClick={() => setActiveCat(c)} style={{ padding: "6px 16px", borderRadius: 20, border: activeCat === c ? "none" : "1px solid #1e1e1e", background: activeCat === c ? "#1e1e1e" : "transparent", color: activeCat === c ? "#D4AF37" : "#444", cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 11, whiteSpace: "nowrap", fontWeight: activeCat === c ? 600 : 400 }}>
                  {c}
                </button>
              ))}
            </div>
          </div>

          {/* Grid */}
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12, padding: 12 }}>
            {filtered.length === 0 && <div style={{ gridColumn: "1/-1", textAlign: "center", padding: "60px 0", color: "#333", fontFamily: "'Montserrat',sans-serif" }}>Bu bo'limda mahsulot yo'q</div>}
            {filtered.map(p => {
              const avg = avgRating(p.reviews);
              return (
                <div key={p.id} onClick={() => { setSelProd(p); setSelSize(""); setScreen("product"); }} style={{ background: "#0a0a0a", borderRadius: 10, overflow: "hidden", border: "1px solid #111", cursor: "pointer", transition: "border-color .2s" }}>
                  <div style={{ height: 170, background: `url(${p.image}) center/cover no-repeat`, position: "relative" }}>
                    <span style={{ position: "absolute", top: 8, left: 8, background: p.gender === "ayol" ? "#a0547a" : "#3a6ea5", color: "#fff", fontSize: 9, padding: "3px 8px", borderRadius: 3, fontFamily: "'Montserrat',sans-serif", fontWeight: 600, letterSpacing: 1 }}>{p.gender === "ayol" ? "♀" : "♂"}</span>
                    <span style={{ position: "absolute", top: 8, right: 8, background: "rgba(212,175,55,0.15)", color: "#D4AF37", fontSize: 9, padding: "3px 8px", borderRadius: 3, fontFamily: "'Montserrat',sans-serif", border: "1px solid #D4AF3744" }}>{p.category}</span>
                  </div>
                  <div style={{ padding: "10px 12px" }}>
                    <div style={{ fontSize: 13, fontWeight: 600, color: "#C0C0C0", lineHeight: 1.3, marginBottom: 4 }}>{p.name}</div>
                    <div style={{ color: "#D4AF37", fontSize: 13, fontWeight: 700 }}>{formatPrice(p.price)}</div>
                    <div style={{ color: "#444", fontSize: 11, fontFamily: "'Montserrat',sans-serif", marginTop: 4 }}>{avg ? `⭐ ${avg}` : "⭐ —"} <span style={{ color: "#333" }}>({p.reviews.length})</span></div>
                  </div>
                </div>
              );
            })}
          </div>
        </div>
      )}

      {/* ══════════════ PRODUCT DETAIL ══════════════ */}
      {screen === "product" && selProd && (
        <div>
          <button onClick={() => setScreen("main")} style={{ position: "fixed", top: 14, left: 14, zIndex: 30, background: "rgba(0,0,0,0.85)", border: "1px solid #D4AF37", color: "#D4AF37", borderRadius: 6, padding: "8px 16px", cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 12, backdropFilter: "blur(4px)" }}>← Orqaga</button>

          <div style={{ height: 300, background: `url(${selProd.image}) center/cover no-repeat`, position: "relative" }}>
            <div style={{ position: "absolute", bottom: 0, left: 0, right: 0, height: 80, background: "linear-gradient(transparent,#000)" }} />
          </div>

          <div style={{ padding: "20px 20px 32px" }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 8 }}>
              <h2 style={{ color: "#C0C0C0", margin: 0, fontSize: 22, flex: 1, lineHeight: 1.3 }}>{selProd.name}</h2>
              <span style={{ color: "#D4AF37", fontSize: 20, fontWeight: 700, marginLeft: 16, whiteSpace: "nowrap" }}>{formatPrice(selProd.price)}</span>
            </div>
            <p style={{ color: "#555", fontFamily: "'Montserrat',sans-serif", fontSize: 13, lineHeight: 1.7, marginBottom: 24 }}>{selProd.description}</p>

            {/* Size */}
            <div style={{ marginBottom: 24 }}>
              <label style={S.label}>RAZMER TANLANG</label>
              <div style={{ display: "flex", gap: 8, flexWrap: "wrap" }}>
                {selProd.sizes.map(sz => (
                  <button key={sz} onClick={() => setSelSize(sz)} style={{ padding: "10px 18px", border: selSize === sz ? "2px solid #D4AF37" : "1px solid #222", background: selSize === sz ? "rgba(212,175,55,0.12)" : "transparent", color: selSize === sz ? "#D4AF37" : "#555", borderRadius: 6, cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontWeight: selSize === sz ? 700 : 400 }}>{sz}</button>
                ))}
              </div>
            </div>

            <button onClick={() => addToCart(selProd, selSize)} style={S.btn(true)}>🛒 SAVATGA QO'SHISH</button>

            {/* Reviews */}
            <div style={{ borderTop: "1px solid #111", paddingTop: 24, marginTop: 28 }}>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 16 }}>
                <h3 style={{ color: "#D4AF37", margin: 0, fontSize: 15, letterSpacing: 1 }}>FIKRLAR ({selProd.reviews.length})</h3>
                <button onClick={() => { setReviewProd(selProd); setScreen("review"); }} style={{ background: "none", border: "1px solid #D4AF37", color: "#D4AF37", padding: "6px 14px", borderRadius: 6, cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 12 }}>+ Fikr</button>
              </div>
              {selProd.reviews.length === 0
                ? <p style={{ color: "#333", fontFamily: "'Montserrat',sans-serif", fontSize: 13, textAlign: "center", padding: "20px 0" }}>Hali fikr yo'q</p>
                : selProd.reviews.map((r, i) => (
                  <div key={i} style={{ background: "#0a0a0a", borderRadius: 8, padding: 14, marginBottom: 10, border: "1px solid #111" }}>
                    <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 6 }}>
                      <span style={{ color: "#D4AF37", fontSize: 13, fontFamily: "'Montserrat',sans-serif", fontWeight: 600 }}>{r.author}</span>
                      <span>{"⭐".repeat(r.rating)}</span>
                    </div>
                    <p style={{ margin: 0, color: "#C0C0C0", fontFamily: "'Montserrat',sans-serif", fontSize: 13, lineHeight: 1.5 }}>{r.text}</p>
                  </div>
                ))}
            </div>
          </div>
        </div>
      )}

      {/* ══════════════ REVIEW ══════════════ */}
      {screen === "review" && (
        <div style={{ padding: 24 }}>
          <button onClick={() => setScreen("product")} style={{ background: "none", border: "none", color: "#D4AF37", cursor: "pointer", marginBottom: 24, fontFamily: "'Montserrat',sans-serif" }}>← Orqaga</button>
          <h2 style={{ color: "#D4AF37", marginBottom: 24, letterSpacing: 1 }}>FIKR QOLDIRING</h2>
          <label style={S.label}>BAHO</label>
          <div style={{ display: "flex", gap: 6, marginBottom: 20 }}>
            {[1, 2, 3, 4, 5].map(n => <button key={n} onClick={() => setReviewRating(n)} style={{ background: "none", border: "none", cursor: "pointer", fontSize: 32, opacity: n <= reviewRating ? 1 : 0.2, padding: 0 }}>⭐</button>)}
          </div>
          <label style={S.label}>FIKRINGIZ</label>
          <textarea placeholder="Mahsulot haqida yozing..." value={reviewText} onChange={e => setReviewText(e.target.value)}
            style={{ ...S.inp, minHeight: 120, resize: "vertical", marginBottom: 20 }} />
          <button onClick={submitReview} style={S.btn(true)}>YUBORISH</button>
        </div>
      )}

      {/* ══════════════ CART ══════════════ */}
      {screen === "cart" && (
        <div style={{ padding: 20 }}>
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 24 }}>
            <button onClick={() => setScreen("main")} style={{ background: "none", border: "none", color: "#D4AF37", cursor: "pointer", fontFamily: "'Montserrat',sans-serif" }}>← Orqaga</button>
            <h2 style={{ color: "#D4AF37", margin: 0, letterSpacing: 1 }}>SAVAT</h2>
            <span style={{ color: "#333", fontFamily: "'Montserrat',sans-serif", fontSize: 12 }}>{totalQty} ta</span>
          </div>

          {cart.length === 0 ? (
            <div style={{ textAlign: "center", padding: "70px 0" }}>
              <div style={{ fontSize: 52, marginBottom: 16 }}>🛒</div>
              <p style={{ color: "#333", fontFamily: "'Montserrat',sans-serif" }}>Savat bo'sh</p>
              <button onClick={() => setScreen("main")} style={{ ...S.btn(false), marginTop: 12 }}>Xarid qilish</button>
            </div>
          ) : (
            <>
              {cart.map((item, i) => (
                <div key={i} style={{ display: "flex", gap: 12, background: "#0a0a0a", border: "1px solid #111", borderRadius: 10, padding: 12, marginBottom: 12 }}>
                  <div style={{ width: 68, height: 68, background: `url(${item.image}) center/cover`, borderRadius: 8, flexShrink: 0 }} />
                  <div style={{ flex: 1 }}>
                    <div style={{ color: "#C0C0C0", fontWeight: 600, fontSize: 13, marginBottom: 3 }}>{item.name}</div>
                    <div style={{ color: "#555", fontSize: 11, fontFamily: "'Montserrat',sans-serif", marginBottom: 4 }}>Razmer: {item.size} · {item.qty} ta</div>
                    <div style={{ color: "#D4AF37", fontWeight: 700 }}>{formatPrice(item.price * item.qty)}</div>
                  </div>
                  <button onClick={() => removeCart(item.id, item.size)} style={{ background: "none", border: "none", color: "#333", cursor: "pointer", fontSize: 18, alignSelf: "flex-start" }}>✕</button>
                </div>
              ))}

              {/* Address select */}
              <div style={{ background: "#0a0a0a", border: "1px solid #D4AF3744", borderRadius: 10, padding: 16, marginTop: 8, marginBottom: 16 }}>
                <label style={S.label}>YETKAZISH MANZILI</label>
                {user?.addresses?.length > 0 && (
                  <div style={{ marginBottom: 12 }}>
                    {user.addresses.map((a, i) => (
                      <label key={i} style={{ display: "flex", alignItems: "center", gap: 10, marginBottom: 8, cursor: "pointer" }}>
                        <input type="radio" name="addr" value={a} checked={selAddr === a} onChange={() => setSelAddr(a)} style={{ accentColor: "#D4AF37" }} />
                        <span style={{ color: "#C0C0C0", fontFamily: "'Montserrat',sans-serif", fontSize: 13 }}>{a}</span>
                      </label>
                    ))}
                    <label style={{ display: "flex", alignItems: "center", gap: 10, cursor: "pointer" }}>
                      <input type="radio" name="addr" value="__manual__" checked={selAddr === "__manual__"} onChange={() => setSelAddr("__manual__")} style={{ accentColor: "#D4AF37" }} />
                      <span style={{ color: "#555", fontFamily: "'Montserrat',sans-serif", fontSize: 13 }}>Yangi manzil kiritish</span>
                    </label>
                  </div>
                )}
                {(selAddr === "__manual__" || user?.addresses?.length === 0) && (
                  <input placeholder="Shahar, ko'cha, uy raqami..." value={manualAddr} onChange={e => setManualAddr(e.target.value)} style={{ ...S.inp }} />
                )}
              </div>

              <div style={{ display: "flex", justifyContent: "space-between", padding: "0 4px", marginBottom: 16 }}>
                <span style={{ color: "#555", fontFamily: "'Montserrat',sans-serif" }}>Jami:</span>
                <span style={{ color: "#D4AF37", fontSize: 20, fontWeight: 700 }}>{formatPrice(totalCart)}</span>
              </div>

              <button onClick={placeOrder} style={S.btn(true)}>✓ BUYURTMA BERISH</button>
            </>
          )}
        </div>
      )}

      {/* ══════════════ PROFILE ══════════════ */}
      {screen === "profile" && user && (
        <div style={{ padding: 20 }}>
          {/* Header */}
          <div style={{ textAlign: "center", padding: "28px 0 32px" }}>
            <div style={{ width: 80, height: 80, background: "linear-gradient(135deg,#D4AF37,#8B6914)", borderRadius: "50%", display: "flex", alignItems: "center", justifyContent: "center", margin: "0 auto 12px", fontSize: 32 }}>👤</div>
            <h2 style={{ color: "#D4AF37", margin: "0 0 4px", fontSize: 22 }}>{user.nickname}</h2>
            <p style={{ color: "#444", margin: 0, fontFamily: "'Montserrat',sans-serif", fontSize: 13 }}>{user.phone}</p>
          </div>

          {/* Info cards */}
          <ProfileCard label="NIKINGIZ" value={user.nickname} onEdit={() => { setEditNick(user.nickname); setProfileEdit("nick"); }} />
          <ProfileCard label="TELEFON" value={user.phone} onEdit={() => { setEditPhone(""); setProfileEdit("phone"); }} />

          {/* Edit forms */}
          {profileEdit === "nick" && (
            <EditBox label="Yangi nik" value={editNick} onChange={setEditNick} onSave={saveNick} onCancel={() => setProfileEdit("")} />
          )}
          {profileEdit === "phone" && (
            <div style={{ background: "#0a0a0a", border: "1px solid #D4AF3744", borderRadius: 10, padding: 16, marginBottom: 16 }}>
              <label style={S.label}>YANGI NOMER (+998...)</label>
              <input value={editPhone} onChange={e => setEditPhone(e.target.value)} style={{ ...S.inp, marginBottom: 12 }} placeholder="901234567" />
              <div style={{ display: "flex", gap: 8 }}>
                <button onClick={savePhone} style={{ ...S.btn(false), flex: 1 }}>Saqlash</button>
                <button onClick={() => setProfileEdit("")} style={{ ...S.ghost, flex: 1 }}>Bekor</button>
              </div>
            </div>
          )}

          {/* Addresses */}
          <div style={{ background: "#0a0a0a", border: "1px solid #111", borderRadius: 10, padding: 16, marginBottom: 16 }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 12 }}>
              <span style={{ ...S.label, margin: 0 }}>MANZILLARIM</span>
              <button onClick={() => setProfileEdit("addr")} style={{ background: "none", border: "1px solid #D4AF37", color: "#D4AF37", padding: "4px 12px", borderRadius: 4, cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 11 }}>+ Qo'shish</button>
            </div>
            {user.addresses.length === 0 && <p style={{ color: "#333", fontFamily: "'Montserrat',sans-serif", fontSize: 12, margin: 0 }}>Manzil yo'q</p>}
            {user.addresses.map((a, i) => (
              <div key={i} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "8px 0", borderBottom: "1px solid #111" }}>
                <span style={{ color: "#C0C0C0", fontFamily: "'Montserrat',sans-serif", fontSize: 13, flex: 1 }}>📍 {a}</span>
                <button onClick={() => removeAddress(i)} style={{ background: "none", border: "none", color: "#333", cursor: "pointer", fontSize: 16 }}>✕</button>
              </div>
            ))}
            {profileEdit === "addr" && (
              <div style={{ marginTop: 12 }}>
                <input placeholder="Shahar, ko'cha, uy..." value={newAddr} onChange={e => setNewAddr(e.target.value)} style={{ ...S.inp, marginBottom: 10 }} />
                <div style={{ display: "flex", gap: 8 }}>
                  <button onClick={addAddress} style={{ ...S.btn(false), flex: 1 }}>Saqlash</button>
                  <button onClick={() => setProfileEdit("")} style={{ ...S.ghost, flex: 1 }}>Bekor</button>
                </div>
              </div>
            )}
          </div>

          {/* My orders */}
          {user.myOrders?.length > 0 && (
            <div style={{ background: "#0a0a0a", border: "1px solid #111", borderRadius: 10, padding: 16, marginBottom: 16 }}>
              <p style={{ ...S.label, marginBottom: 12 }}>MENING BUYURTMALARIM</p>
              {user.myOrders.map(o => (
                <div key={o.id} style={{ borderBottom: "1px solid #111", paddingBottom: 10, marginBottom: 10 }}>
                  <div style={{ display: "flex", justifyContent: "space-between" }}>
                    <span style={{ color: "#D4AF37", fontFamily: "'Montserrat',sans-serif", fontSize: 12 }}>#{o.id.toString().slice(-5)}</span>
                    <span style={{ color: "#333", fontFamily: "'Montserrat',sans-serif", fontSize: 11 }}>{o.date}</span>
                  </div>
                  <div style={{ color: "#555", fontFamily: "'Montserrat',sans-serif", fontSize: 12, marginTop: 4 }}>{o.items.map(i => i.name).join(", ")}</div>
                  <div style={{ color: "#D4AF37", fontFamily: "'Montserrat',sans-serif", fontSize: 13, fontWeight: 700, marginTop: 4 }}>{formatPrice(o.total)}</div>
                </div>
              ))}
            </div>
          )}

          {/* Support */}
          <div style={{ background: "#0a0a0a", border: "1px solid #111", borderRadius: 10, overflow: "hidden", marginBottom: 16 }}>
            <p style={{ ...S.label, margin: "16px 16px 12px" }}>QO'LLAB-QUVVATLASH</p>
            <a href={`https://t.me/${SUPPORT_TG}`} target="_blank" rel="noopener noreferrer"
              style={{ display: "flex", alignItems: "center", gap: 12, padding: "14px 16px", background: "linear-gradient(135deg,#0088cc22,#0088cc11)", borderTop: "1px solid #111", textDecoration: "none" }}>
              <span style={{ fontSize: 24 }}>💬</span>
              <div>
                <div style={{ color: "#0088cc", fontFamily: "'Montserrat',sans-serif", fontWeight: 600, fontSize: 14 }}>24/7 Qo'llab-quvvatlash</div>
                <div style={{ color: "#444", fontFamily: "'Montserrat',sans-serif", fontSize: 12 }}>@{SUPPORT_TG} ga murojaat qiling</div>
              </div>
              <span style={{ marginLeft: "auto", color: "#333", fontSize: 18 }}>→</span>
            </a>
            <a href={`https://t.me/${SUPPORT_TG}?text=Salom%20admin%2C%20sizga%20murojaat%20qilmoqchiman`} target="_blank" rel="noopener noreferrer"
              style={{ display: "flex", alignItems: "center", gap: 12, padding: "14px 16px", borderTop: "1px solid #111", textDecoration: "none" }}>
              <span style={{ fontSize: 24 }}>📨</span>
              <div>
                <div style={{ color: "#D4AF37", fontFamily: "'Montserrat',sans-serif", fontWeight: 600, fontSize: 14 }}>Adminga murojaat</div>
                <div style={{ color: "#444", fontFamily: "'Montserrat',sans-serif", fontSize: 12 }}>Telegram orqali avtomatik xabar yuboriladi</div>
              </div>
              <span style={{ marginLeft: "auto", color: "#333", fontSize: 18 }}>→</span>
            </a>
          </div>

          <button onClick={() => { setUser(null); setCart([]); setScreen("auth"); setAuthStep("phone"); setAuthPhone(""); setAuthNick(""); }}
            style={{ width: "100%", padding: "14px", background: "transparent", border: "1px solid #1e1e1e", color: "#444", borderRadius: 8, cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 13 }}>
            Chiqish
          </button>
        </div>
      )}

      {/* ══════════════ SUPPORT ══════════════ */}
      {screen === "support" && (
        <div style={{ padding: 24 }}>
          <button onClick={() => setScreen("main")} style={{ background: "none", border: "none", color: "#D4AF37", cursor: "pointer", marginBottom: 28, fontFamily: "'Montserrat',sans-serif" }}>← Orqaga</button>
          <div style={{ textAlign: "center", marginBottom: 36 }}>
            <div style={{ fontSize: 52, marginBottom: 12 }}>🎧</div>
            <h2 style={{ color: "#D4AF37", margin: "0 0 8px", letterSpacing: 2 }}>24/7 SUPPORT</h2>
            <p style={{ color: "#555", fontFamily: "'Montserrat',sans-serif", fontSize: 13 }}>Istalgan vaqt bog'laning</p>
          </div>

          {[
            { icon: "💬", title: "Jonli chat", desc: "Tez javob, 24/7", link: `https://t.me/${SUPPORT_TG}`, color: "#0088cc" },
            { icon: "📨", title: "Adminga yozish", desc: "Muammolaringizni yuboring", link: `https://t.me/${SUPPORT_TG}?text=Salom%20admin%2C%20sizga%20murojaat%20qilmoqchiman`, color: "#D4AF37" },
          ].map((item, i) => (
            <a key={i} href={item.link} target="_blank" rel="noopener noreferrer"
              style={{ display: "flex", alignItems: "center", gap: 16, background: "#0a0a0a", border: `1px solid ${item.color}33`, borderRadius: 12, padding: 18, marginBottom: 12, textDecoration: "none" }}>
              <div style={{ width: 48, height: 48, background: `${item.color}22`, borderRadius: 10, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 22 }}>{item.icon}</div>
              <div style={{ flex: 1 }}>
                <div style={{ color: item.color, fontFamily: "'Montserrat',sans-serif", fontWeight: 700, fontSize: 15 }}>{item.title}</div>
                <div style={{ color: "#444", fontFamily: "'Montserrat',sans-serif", fontSize: 12, marginTop: 2 }}>{item.desc}</div>
              </div>
              <span style={{ color: "#333", fontSize: 20 }}>→</span>
            </a>
          ))}

          <div style={{ background: "#0a0a0a", border: "1px solid #111", borderRadius: 12, padding: 18, marginTop: 8 }}>
            <p style={{ color: "#D4AF37", fontFamily: "'Montserrat',sans-serif", fontWeight: 600, marginBottom: 8 }}>Ish vaqti</p>
            <p style={{ color: "#555", fontFamily: "'Montserrat',sans-serif", fontSize: 13, lineHeight: 1.8, margin: 0 }}>
              🕐 24 soat, 7 kun<br />
              📍 O'zbekiston<br />
              ⚡ O'rtacha javob: 5 daqiqa
            </p>
          </div>
        </div>
      )}

      {/* ══════════════ BOTTOM NAV ══════════════ */}
      {["main", "cart", "profile", "support"].includes(screen) && (
        <div style={{ position: "fixed", bottom: 0, left: "50%", transform: "translateX(-50%)", width: "100%", maxWidth: 480, background: "#050505", borderTop: "1px solid #111", display: "flex", zIndex: 40 }}>
          {[
            { icon: "🏠", label: "Bosh sahifa", sc: "main" },
            { icon: "🛒", label: "Savat", sc: "cart", badge: totalQty },
            { icon: "👤", label: "Profil", sc: "profile" },
            { icon: "🎧", label: "Support", sc: "support" },
          ].map(item => (
            <button key={item.sc} onClick={() => setScreen(item.sc)} style={{ flex: 1, background: "none", border: "none", color: screen === item.sc ? "#D4AF37" : "#333", cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 10, padding: "10px 0 12px", position: "relative", letterSpacing: 0.3 }}>
              <div style={{ fontSize: 22, marginBottom: 2 }}>{item.icon}</div>
              {item.label}
              {item.badge > 0 && <span style={{ position: "absolute", top: 6, right: "calc(50% - 18px)", background: "#D4AF37", color: "#000", borderRadius: "50%", width: 16, height: 16, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 9, fontWeight: 700 }}>{item.badge}</span>}
            </button>
          ))}
        </div>
      )}

      {/* ══════════════ ADMIN PANEL (full screen overlay) ══════════════ */}
      {screen === "adminEntry" && adminScreen === "panel" && (
        <AdminPanel
          products={products} orders={orders} newProd={newProd} setNewProd={setNewProd}
          editProd={editProd} setEditProd={setEditProd} imgFiles={imgFiles}
          handleFileUpload={handleFileUpload} handleAddProd={handleAddProd}
          handleDeleteProd={handleDeleteProd} handleEditSave={handleEditSave}
          adminTab={adminTab} setAdminTab={setAdminTab} formatPrice={formatPrice}
          fileRef={fileRef} editFileRef={editFileRef}
          onLogout={() => { setAdminScreen("login"); setAdminUser(""); setAdminPass(""); setScreen("auth"); }}
          showNotif={showNotif}
        />
      )}
    </div>
  );
}

// ── PROFILE CARD ───────────────────────────────────────────
function ProfileCard({ label, value, onEdit }) {
  return (
    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", background: "#0a0a0a", border: "1px solid #111", borderRadius: 10, padding: "14px 16px", marginBottom: 10 }}>
      <div>
        <div style={{ color: "#D4AF37", fontSize: 10, letterSpacing: 2, fontFamily: "'Montserrat',sans-serif", marginBottom: 4 }}>{label}</div>
        <div style={{ color: "#C0C0C0", fontFamily: "'Montserrat',sans-serif", fontSize: 14 }}>{value}</div>
      </div>
      <button onClick={onEdit} style={{ background: "none", border: "1px solid #222", color: "#555", padding: "6px 14px", borderRadius: 6, cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 11 }}>✏️</button>
    </div>
  );
}

function EditBox({ label, value, onChange, onSave, onCancel }) {
  return (
    <div style={{ background: "#0a0a0a", border: "1px solid #D4AF3744", borderRadius: 10, padding: 16, marginBottom: 16 }}>
      <label style={{ display: "block", color: "#D4AF37", fontSize: 11, letterSpacing: 2, marginBottom: 8, fontFamily: "'Montserrat',sans-serif" }}>{label.toUpperCase()}</label>
      <input value={value} onChange={e => onChange(e.target.value)} style={{ width: "100%", padding: "12px 16px", background: "#000", border: "1px solid #2a2a2a", borderRadius: 6, color: "#C0C0C0", fontFamily: "'Montserrat',sans-serif", fontSize: 14, outline: "none", boxSizing: "border-box", marginBottom: 12 }} />
      <div style={{ display: "flex", gap: 8 }}>
        <button onClick={onSave} style={{ flex: 1, padding: "12px", background: "linear-gradient(135deg,#D4AF37,#b8960c)", color: "#000", border: "none", borderRadius: 6, fontWeight: 700, cursor: "pointer", fontFamily: "'Montserrat',sans-serif" }}>Saqlash</button>
        <button onClick={onCancel} style={{ flex: 1, padding: "12px", background: "transparent", color: "#555", border: "1px solid #1e1e1e", borderRadius: 6, cursor: "pointer", fontFamily: "'Montserrat',sans-serif" }}>Bekor</button>
      </div>
    </div>
  );
}

// ── ADMIN PANEL COMPONENT ──────────────────────────────────
function AdminPanel({ products, orders, newProd, setNewProd, editProd, setEditProd, imgFiles, handleFileUpload, handleAddProd, handleDeleteProd, handleEditSave, adminTab, setAdminTab, formatPrice, fileRef, editFileRef, onLogout, showNotif }) {
  const inp = { width: "100%", padding: "11px 14px", background: "#0d0d0d", border: "1px solid #2a2a2a", borderRadius: 6, color: "#C0C0C0", fontFamily: "'Montserrat',sans-serif", fontSize: 13, outline: "none", boxSizing: "border-box" };
  const lbl = { display: "block", color: "#D4AF37", fontSize: 10, letterSpacing: 2, marginBottom: 6, fontFamily: "'Montserrat',sans-serif" };

  return (
    <div style={{ minHeight: "100vh", background: "#000", color: "#C0C0C0", maxWidth: 480, margin: "0 auto", paddingBottom: 40 }}>
      {/* Admin header */}
      <div style={{ background: "#0a0a0a", borderBottom: "1px solid #111", padding: "16px 20px", display: "flex", justifyContent: "space-between", alignItems: "center" }}>
        <div>
          <h2 style={{ color: "#D4AF37", margin: 0, fontSize: 18, letterSpacing: 2 }}>🛡️ ADMIN</h2>
          <p style={{ color: "#333", margin: 0, fontFamily: "'Montserrat',sans-serif", fontSize: 11 }}>fazilov · LUXE Panel</p>
        </div>
        <button onClick={onLogout} style={{ background: "none", border: "1px solid #1e1e1e", color: "#444", padding: "8px 14px", borderRadius: 6, cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 12 }}>Chiqish</button>
      </div>

      {/* Stats */}
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 8, padding: "16px 20px 8px" }}>
        {[
          { label: "Buyurtmalar", val: orders.length, icon: "📦" },
          { label: "Mahsulotlar", val: products.length, icon: "👗" },
          { label: "Jami summa", val: orders.reduce((s, o) => s + o.total, 0), icon: "💰", money: true },
        ].map((s, i) => (
          <div key={i} style={{ background: "#0a0a0a", border: "1px solid #111", borderRadius: 10, padding: "12px 10px", textAlign: "center" }}>
            <div style={{ fontSize: 20, marginBottom: 4 }}>{s.icon}</div>
            <div style={{ color: "#D4AF37", fontWeight: 700, fontSize: s.money ? 10 : 18 }}>{s.money ? formatPrice(s.val) : s.val}</div>
            <div style={{ color: "#333", fontFamily: "'Montserrat',sans-serif", fontSize: 10, marginTop: 2 }}>{s.label}</div>
          </div>
        ))}
      </div>

      {/* Tabs */}
      <div style={{ display: "flex", gap: 8, padding: "8px 20px 16px" }}>
        {["orders", "products", "add"].map(tab => (
          <button key={tab} onClick={() => setAdminTab(tab)} style={{ flex: 1, padding: "10px 4px", background: adminTab === tab ? "#D4AF37" : "#0a0a0a", color: adminTab === tab ? "#000" : "#444", border: adminTab === tab ? "none" : "1px solid #111", borderRadius: 6, cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 11, fontWeight: adminTab === tab ? 700 : 400, letterSpacing: 0.5 }}>
            {tab === "orders" ? "📦 Buyurtma" : tab === "products" ? "👗 Mahsulot" : "➕ Qo'shish"}
          </button>
        ))}
      </div>

      <div style={{ padding: "0 20px" }}>
        {/* ORDERS TAB */}
        {adminTab === "orders" && (
          <div>
            {orders.length === 0 && <div style={{ textAlign: "center", padding: "50px 0", color: "#333", fontFamily: "'Montserrat',sans-serif" }}>Hali buyurtma yo'q</div>}
            {orders.map(o => (
              <div key={o.id} style={{ background: "#0a0a0a", border: "1px solid #D4AF3733", borderRadius: 10, padding: 16, marginBottom: 12 }}>
                <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 10 }}>
                  <span style={{ color: "#D4AF37", fontWeight: 700, fontFamily: "'Montserrat',sans-serif" }}>#{o.id.toString().slice(-6)}</span>
                  <span style={{ color: "#333", fontFamily: "'Montserrat',sans-serif", fontSize: 11 }}>{o.date}</span>
                </div>
                <div style={{ fontFamily: "'Montserrat',sans-serif", fontSize: 13, lineHeight: 2 }}>
                  <div>👤 <span style={{ color: "#C0C0C0" }}>{o.nickname}</span></div>
                  <div>📞 <span style={{ color: "#D4AF37", fontWeight: 600 }}>{o.phone}</span></div>
                  <div>📍 <span style={{ color: "#C0C0C0" }}>{o.address}</span></div>
                </div>
                <div style={{ borderTop: "1px solid #111", marginTop: 10, paddingTop: 10 }}>
                  {o.items.map((item, i) => (
                    <div key={i} style={{ color: "#555", fontFamily: "'Montserrat',sans-serif", fontSize: 12, marginBottom: 3 }}>
                      • {item.name} ({item.size}) × {item.qty} — {formatPrice(item.price * item.qty)}
                    </div>
                  ))}
                  <div style={{ color: "#D4AF37", fontWeight: 700, fontFamily: "'Montserrat',sans-serif", marginTop: 8, fontSize: 14 }}>Jami: {formatPrice(o.total)}</div>
                </div>
              </div>
            ))}
          </div>
        )}

        {/* PRODUCTS TAB */}
        {adminTab === "products" && (
          <div>
            {products.map(p => (
              <div key={p.id}>
                {editProd?.id === p.id ? (
                  <div style={{ background: "#0a0a0a", border: "1px solid #D4AF37", borderRadius: 10, padding: 16, marginBottom: 12 }}>
                    <p style={{ color: "#D4AF37", fontFamily: "'Montserrat',sans-serif", fontSize: 12, letterSpacing: 1, marginBottom: 14 }}>✏️ TAHRIRLASH</p>
                    {[
                      { key: "name", label: "Nomi", ph: "Mahsulot nomi" },
                      { key: "price", label: "Narx", ph: "250000", type: "number" },
                      { key: "description", label: "Tavsif", ph: "..." },
                      { key: "sizes", label: "Razmerlar (vergul)", ph: "S, M, L, XL" },
                    ].map(f => (
                      <div key={f.key} style={{ marginBottom: 10 }}>
                        <label style={lbl}>{f.label.toUpperCase()}</label>
                        <input type={f.type || "text"} placeholder={f.ph}
                          value={typeof editProd[f.key] === "object" ? editProd[f.key].join(", ") : editProd[f.key]}
                          onChange={e => setEditProd({ ...editProd, [f.key]: e.target.value })}
                          style={inp} />
                      </div>
                    ))}
                    <div style={{ marginBottom: 10 }}>
                      <label style={lbl}>JINSI</label>
                      <select value={editProd.gender} onChange={e => setEditProd({ ...editProd, gender: e.target.value })} style={{ ...inp }}>
                        <option value="ayol">Ayol</option>
                        <option value="erkak">Erkak</option>
                      </select>
                    </div>
                    <div style={{ marginBottom: 14 }}>
                      <label style={lbl}>RASM (Galareya)</label>
                      <input type="file" accept="image/*" ref={editFileRef} style={{ display: "none" }} onChange={e => handleFileUpload(e, "edit_" + editProd.id)} />
                      <button onClick={() => editFileRef.current?.click()} style={{ width: "100%", padding: "12px", background: "#111", border: "1px dashed #333", borderRadius: 6, color: "#555", cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 12 }}>
                        {imgFiles["edit_" + editProd.id] ? "✅ Rasm tanlandi" : "📷 Galareya"}
                      </button>
                    </div>
                    <div style={{ display: "flex", gap: 8 }}>
                      <button onClick={handleEditSave} style={{ flex: 1, padding: "12px", background: "#D4AF37", color: "#000", border: "none", borderRadius: 6, fontWeight: 700, cursor: "pointer", fontFamily: "'Montserrat',sans-serif" }}>Saqlash</button>
                      <button onClick={() => setEditProd(null)} style={{ flex: 1, padding: "12px", background: "transparent", color: "#555", border: "1px solid #111", borderRadius: 6, cursor: "pointer", fontFamily: "'Montserrat',sans-serif" }}>Bekor</button>
                    </div>
                  </div>
                ) : (
                  <div style={{ display: "flex", gap: 12, background: "#0a0a0a", border: "1px solid #111", borderRadius: 10, padding: 12, marginBottom: 10 }}>
                    <div style={{ width: 64, height: 64, background: `url(${p.image}) center/cover`, borderRadius: 8, flexShrink: 0 }} />
                    <div style={{ flex: 1, minWidth: 0 }}>
                      <div style={{ color: "#C0C0C0", fontWeight: 600, fontSize: 13, marginBottom: 2 }}>{p.name}</div>
                      <div style={{ color: "#D4AF37", fontSize: 12, fontFamily: "'Montserrat',sans-serif" }}>{formatPrice(p.price)}</div>
                      <div style={{ color: "#333", fontSize: 11, fontFamily: "'Montserrat',sans-serif" }}>{p.gender === "ayol" ? "♀ Ayol" : "♂ Erkak"} · {p.category}</div>
                    </div>
                    <div style={{ display: "flex", flexDirection: "column", gap: 6 }}>
                      <button onClick={() => setEditProd({ ...p, sizes: p.sizes.join(", ") })} style={{ background: "#D4AF3722", border: "1px solid #D4AF3744", color: "#D4AF37", padding: "6px 10px", borderRadius: 6, cursor: "pointer", fontSize: 12 }}>✏️</button>
                      <button onClick={() => handleDeleteProd(p.id)} style={{ background: "#ff444422", border: "1px solid #ff444444", color: "#ff4444", padding: "6px 10px", borderRadius: 6, cursor: "pointer", fontSize: 12 }}>🗑</button>
                    </div>
                  </div>
                )}
              </div>
            ))}
          </div>
        )}

        {/* ADD PRODUCT TAB */}
        {adminTab === "add" && (
          <div>
            <p style={{ color: "#D4AF37", fontFamily: "'Montserrat',sans-serif", fontSize: 12, letterSpacing: 2, marginBottom: 20 }}>YANGI MAHSULOT</p>
            {[
              { key: "name", label: "Mahsulot nomi *", ph: "Charm sumka" },
              { key: "price", label: "Narxi (so'm) *", ph: "250000", type: "number" },
              { key: "description", label: "Tavsif", ph: "Mahsulot haqida..." },
              { key: "sizes", label: "Razmerlar (vergul bilan)", ph: "XS, S, M, L, XL" },
            ].map(f => (
              <div key={f.key} style={{ marginBottom: 14 }}>
                <label style={lbl}>{f.label.toUpperCase()}</label>
                <input type={f.type || "text"} placeholder={f.ph} value={newProd[f.key]} onChange={e => setNewProd({ ...newProd, [f.key]: e.target.value })} style={inp} />
              </div>
            ))}

            <div style={{ marginBottom: 14 }}>
              <label style={lbl}>KATEGORIYA</label>
              <select value={newProd.category} onChange={e => setNewProd({ ...newProd, category: e.target.value })} style={inp}>
                <option>Kiyimlar</option><option>Sumkalar</option><option>Aksesuar</option>
              </select>
            </div>

            <div style={{ marginBottom: 14 }}>
              <label style={lbl}>JINSI</label>
              <select value={newProd.gender} onChange={e => setNewProd({ ...newProd, gender: e.target.value })} style={inp}>
                <option value="ayol">Ayol</option><option value="erkak">Erkak</option>
              </select>
            </div>

            <div style={{ marginBottom: 20 }}>
              <label style={lbl}>RASM (GALAREYA)</label>
              <input type="file" accept="image/*" ref={fileRef} style={{ display: "none" }} onChange={e => handleFileUpload(e, "new_" + Date.now())} />
              <button onClick={() => fileRef.current?.click()} style={{ width: "100%", padding: "40px 20px", background: "#0a0a0a", border: "2px dashed #222", borderRadius: 10, color: "#333", cursor: "pointer", fontFamily: "'Montserrat',sans-serif", fontSize: 13, textAlign: "center" }}>
                {Object.keys(imgFiles).some(k => k.startsWith("new_"))
                  ? <span style={{ color: "#D4AF37" }}>✅ Rasm tanlandi</span>
                  : <><div style={{ fontSize: 32, marginBottom: 8 }}>📷</div>Galereyadan rasm tanlang</>}
              </button>
            </div>

            <button onClick={handleAddProd} style={{ width: "100%", padding: "16px", background: "linear-gradient(135deg,#D4AF37,#b8960c)", color: "#000", border: "none", borderRadius: 8, fontWeight: 700, fontSize: 14, letterSpacing: 2, fontFamily: "'Montserrat',sans-serif", cursor: "pointer" }}>
              ✓ MAHSULOT QO'SHISH
            </button>
          </div>
        )}
      </div>
    </div>
  );
} 

shu kodni telegram botmga ulamoqchiman nima qilishim kerak va uni qanday serverga joylayman