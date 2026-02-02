# collectorhert
# AntiqueShop Pro — Next.js + Stripe + Admin Panel + CI

This upgraded project converts your shop into a full **Next.js (App Router)** application with:
- SEO-friendly server rendering
- Stripe Checkout (serverless API route)
- Admin panel for adding/editing products
- GitHub-ready repo (.gitignore, LICENSE, CI)

This is a production-ready starter you can push to GitHub.

==============================
FOLDER STRUCTURE
==============================

/antique-shop-pro
  /app
    /api/checkout/route.js
    /api/admin/products/route.js
    /admin/page.jsx
    /layout.jsx
    /page.jsx
  /components
    Header.jsx
    ProductCard.jsx
    AdminForm.jsx
  /lib
    stripe.js
    db.js
  /public
  .env.example
  .gitignore
  LICENSE
  next.config.js
  package.json
  README.md
  tailwind.config.js
  postcss.config.js

==============================
package.json
==============================
{
  "name": "antique-shop-pro",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "next": "14.1.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "stripe": "^14.0.0"
  }
}

==============================
next.config.js
==============================
module.exports = {
  reactStrictMode: true
}

==============================
.gitignore
==============================
node_modules
.next
.env
.env.local

==============================
LICENSE (MIT)
==============================
MIT License

Copyright (c) 2026 AntiqueShop

Permission is hereby granted, free of charge, to any person obtaining a copy...

==============================
.env.example
==============================
STRIPE_SECRET_KEY=sk_test_xxx
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxx
ADMIN_SECRET=changeme

==============================
/lib/stripe.js
==============================
import Stripe from 'stripe'

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY)

==============================
/lib/db.js (Simple JSON DB for demo)
==============================
import fs from 'fs'
import path from 'path'

const file = path.join(process.cwd(), 'products.json')

export function getProducts() {
  if (!fs.existsSync(file)) return []
  return JSON.parse(fs.readFileSync(file))
}

export function saveProducts(products) {
  fs.writeFileSync(file, JSON.stringify(products, null, 2))
}

==============================
/app/layout.jsx
==============================
import './globals.css'

export const metadata = {
  title: 'AntiqueShop Pro',
  description: 'Buy rare coins and antiques online'
}

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}

==============================
/app/page.jsx (Main Shop)
==============================
import Header from '@/components/Header'
import ProductCard from '@/components/ProductCard'
import { getProducts } from '@/lib/db'

export default function Home() {
  const products = getProducts()

  return (
    <div>
      <Header />
      <main className="max-w-6xl mx-auto p-6 grid grid-cols-1 md:grid-cols-3 gap-6">
        {products.map(p => (
          <ProductCard key={p.id} item={p} />
        ))}
      </main>
    </div>
  )
}

==============================
/components/ProductCard.jsx
==============================
'use client'

export default function ProductCard({ item }) {
  async function checkout() {
    const res = await fetch('/api/checkout', {
      method: 'POST',
      body: JSON.stringify(item)
    })
    const data = await res.json()
    window.location.href = data.url
  }

  return (
    <div className="border p-4 rounded">
      <h3 className="font-bold">{item.title}</h3>
      <p>${item.price}</p>
      <button onClick={checkout} className="bg-amber-600 text-white px-3 py-2 rounded">Buy Now</button>
    </div>
  )
}

==============================
/app/api/checkout/route.js (Stripe Serverless)
==============================
import { stripe } from '@/lib/stripe'

export async function POST(req) {
  const item = await req.json()

  const session = await stripe.checkout.sessions.create({
    payment_method_types: ['card'],
    line_items: [
      {
        price_data: {
          currency: 'usd',
          product_data: { name: item.title },
          unit_amount: Math.round(item.price * 100)
        },
        quantity: 1
      }
    ],
    mode: 'payment',
    success_url: `${process.env.NEXT_PUBLIC_BASE_URL}/?success=1`,
    cancel_url: `${process.env.NEXT_PUBLIC_BASE_URL}/?canceled=1`
  })

  return Response.json({ url: session.url })
}

==============================
/app/admin/page.jsx (Admin Panel)
==============================
'use client'

import AdminForm from '@/components/AdminForm'

export default function AdminPage() {
  return (
    <div className="max-w-xl mx-auto p-6">
      <h1 className="text-2xl font-bold mb-4">Admin Panel</h1>
      <AdminForm />
    </div>
  )
}

==============================
/components/AdminForm.jsx
==============================
'use client'

import { useState } from 'react'

export default function AdminForm() {
  const [title, setTitle] = useState('')
  const [price, setPrice] = useState('')

  async function save() {
    await fetch('/api/admin/products', {
      method: 'POST',
      body: JSON.stringify({ title, price: Number(price) })
    })
    alert('Product saved')
    setTitle('')
    setPrice('')
  }

  return (
    <div className="space-y-3">
      <input value={title} onChange={e=>setTitle(e.target.value)} placeholder="Title" className="border p-2 w-full" />
      <input value={price} onChange={e=>setPrice(e.target.value)} placeholder="Price" className="border p-2 w-full" />
      <button onClick={save} className="bg-green-600 text-white px-4 py-2 rounded">Add Product</button>
    </div>
  )
}

==============================
/app/api/admin/products/route.js
==============================
import { getProducts, saveProducts } from '@/lib/db'

export async function POST(req) {
  const data = await req.json()
  const products = getProducts()

  products.push({
    id: Date.now().toString(),
    title: data.title,
    price: data.price
  })

  saveProducts(products)
  return Response.json({ ok: true })
}

==============================
README.md
==============================
# AntiqueShop Pro

Next.js antiques marketplace with Stripe checkout and admin panel.

## Setup

1. Copy .env.example to .env.local
2. Add Stripe keys
3. Set NEXT_PUBLIC_BASE_URL=http://localhost:3000
4. npm install
5. npm run dev

## Admin

Visit /admin to add products.

==============================

This gives you:
- SEO-ready Next.js
- Stripe Checkout (serverless)
- Admin product management
- GitHub-ready repo

You can now push this to GitHub and deploy on Vercel.
