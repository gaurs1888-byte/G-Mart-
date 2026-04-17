# G-Mart-
Create a quick commerce store 
:root {
  --bg: #0f0f1a;
  --card: rgba(255,255,255,0.06);
  --accent: #7C6FE9;
  --success: #34D399;
}

body {
  margin: 0;
  font-family: 'Inter', sans-serif;
  background: var(--bg);
  color: white;
}

.container {
  padding: 12px;
}

.card {
  background: var(--card);
  backdrop-filter: blur(16px);
  border-radius: 16px;
  padding: 12px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.3);
}

.button {
  background: var(--accent);
  border: none;
  padding: 10px;
  border-radius: 12px;
  color: white;
  font-weight: 600;
}

.search {
  width: 100%;
  padding: 12px;
  border-radius: 12px;
  border: none;
  background: #1a1a2e;
  color: white;
}
import { useState } from "react";

export default function Header({ onSearch }: any) {
  return (
    <div style={{
      position: "sticky",
      top: 0,
      zIndex: 10,
      background: "#0f0f1a",
      padding: "10px"
    }}>
      <h2>🛒 G-Mart</h2>

      <input
        className="search"
        placeholder="Search products..."
        onChange={(e) => onSearch(e.target.value)}
      />
    </div>
  );
}export default function CategoryFilter({ categories, active, setActive }: any) {
  return (
    <div style={{
      display: "flex",
      overflowX: "auto",
      gap: 10,
      padding: "10px 0"
    }}>
      {["All", ...categories].map(cat => (
        <button
          key={cat}
          onClick={() => setActive(cat)}
          style={{
            padding: "8px 14px",
            borderRadius: 20,
            border: "none",
            background: active === cat ? "#7C6FE9" : "#1a1a2e",
            color: "white",
            whiteSpace: "nowrap"
          }}
        >
          {cat}
        </button>
      ))}
    </div>
  );
}import { useCartStore } from "../store/useCartStore";

export default function ProductCard({ product }: any) {
  const add = useCartStore(s => s.add);

  return (
    <div className="card">
      <div style={{ fontSize: 40 }}>{product.emoji}</div>

      <h4>{product.name}</h4>
      <p style={{ opacity: 0.6 }}>{product.desc}</p>

      <div style={{
        display: "flex",
        justifyContent: "space-between",
        alignItems: "center"
      }}>
        <b>₹{product.price}</b>

        <button className="button" onClick={() => add(product)}>
          ADD
        </button>
      </div>
    </div>
  );
}import { useCartStore } from "../store/useCartStore";
import { useNavigate } from "react-router-dom";

export default function CartButton() {
  const { items, total } = useCartStore();
  const navigate = useNavigate();

  if (!items.length) return null;

  return (
    <div
      onClick={() => navigate("/checkout")}
      style={{
        position: "fixed",
        bottom: 20,
        left: 10,
        right: 10,
        background: "#34D399",
        borderRadius: 14,
        padding: 14,
        display: "flex",
        justifyContent: "space-between",
        color: "black",
        fontWeight: "bold"
      }}
    >
      <span>{items.length} items</span>
      <span>₹{total()}</span>
    </div>
  );
}import { useEffect, useState } from "react";
import { api } from "../api/client";
import ProductCard from "../components/ProductCard";
import CartButton from "../components/CartButton";
import Header from "../components/Header";
import CategoryFilter from "../components/CategoryFilter";

export default function Home() {
  const [products, setProducts] = useState<any[]>([]);
  const [filtered, setFiltered] = useState<any[]>([]);
  const [category, setCategory] = useState("All");

  useEffect(() => {
    api.get("/api/catalog").then(res => {
      setProducts(res.data);
      setFiltered(res.data);
    });
  }, []);

  const categories = [...new Set(products.map(p => p.category))];

  const filterCategory = (cat: string) => {
    setCategory(cat);
    if (cat === "All") return setFiltered(products);
    setFiltered(products.filter(p => p.category === cat));
  };

  const search = (q: string) => {
    setFiltered(
      products.filter(p =>
        p.name.toLowerCase().includes(q.toLowerCase())
      )
    );
  };

  return (
    <div className="container">
      <Header onSearch={search} />

      <CategoryFilter
        categories={categories}
        active={category}
        setActive={filterCategory}
      />

      <div style={{
        display: "grid",
        gridTemplateColumns: "repeat(2,1fr)",
        gap: 10
      }}>
        {filtered.map(p => (
          <ProductCard key={p.id} product={p} />
        ))}
      </div>

      <CartButton />
    </div>
  );
}import { useCartStore } from "../store/useCartStore";

export default function Checkout() {
  const { items, total } = useCartStore();

  return (
    <div className="container">
      <h2>Order Summary</h2>

      {items.map(i => (
        <div className="card" key={i.id}>
          {i.name} x{i.quantity}
        </div>
      ))}

      <h3>Total: ₹{total()}</h3>

      <button className="button">Proceed</button>
    </div>
  );
}
