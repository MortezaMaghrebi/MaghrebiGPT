# MaghrebiGPT

A clean, modern, fully client-side chat UI powered by **Groq API**, protected using a **Cloudflare Worker proxy** to hide your API key.
Live Demo: **[https://mortezamaghrebi.github.io/MaghrebiGPT/](https://mortezamaghrebi.github.io/MaghrebiGPT/)**

---

## 🚀 About the Project

**MaghrebiGPT** is a lightweight ChatGPT-style interface designed to interact with Groq’s ultra-fast LLMs.
All requests are sent through a secure **Cloudflare Worker** so the real `GROQ_API_KEY` is never exposed in the browser.

Features include:

* ✔ Modern ChatGPT-like UI
* ✔ Model selector with multiple Groq-supported models
* ✔ RTL/LTR toggle
* ✔ Syntax highlighting + Markdown support
* ✔ Code block copy/download
* ✔ HTML preview for generated code
* ✔ Chat history stored in browser (localStorage)
* ✔ Fully static — works on GitHub Pages
* ✔ Secure API key protection via Cloudflare Worker

---

## 🧩 How It Works

The frontend (HTML/JS) sends POST requests to your Cloudflare Worker:

```
Frontend  →  Cloudflare Worker  →  Groq API
```

The Worker injects the real `GROQ_API_KEY`, so the browser never sees it.

---

## 🛠️ Installation

Clone the repository:

```bash
git clone https://github.com/MortezaMaghrebi/MaghrebiGPT.git
cd MaghrebiGPT
```

Since it's a static project, you can open `index.html` directly or host it with any static hosting provider (GitHub Pages, Netlify, Vercel, Cloudflare Pages, etc).

---

## 🌐 Cloudflare Worker (API Proxy)

To hide your **Groq API key**, deploy this Worker at Cloudflare:

### **worker.js**

```js
export default {
  async fetch(request, env, ctx) {
    const corsHeaders = {
      'Access-Control-Allow-Origin': 'https://mortezamaghrebi.github.io',
      'Access-Control-Allow-Methods': 'POST, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type',
      'Access-Control-Max-Age': '86400',
    };

    if (request.method === 'OPTIONS') {
      return new Response(null, { headers: corsHeaders });
    }

    if (request.method !== 'POST') {
      return new Response('Method Not Allowed', { status: 405, headers: corsHeaders });
    }

    try {
      const body = await request.json();
      const apiKey = env.GROQ_API_KEY;

      if (!apiKey) {
        return new Response(JSON.stringify({ error: 'API key is missing' }), {
          status: 500,
          headers: { ...corsHeaders, 'Content-Type': 'application/json' }
        });
      }

      const groqResponse = await fetch('https://api.groq.com/openai/v1/chat/completions', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${apiKey}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(body)
      });

      const groqData = await groqResponse.json();

      return new Response(JSON.stringify(groqData), {
        status: groqResponse.status,
        headers: {
          ...corsHeaders,
          'Content-Type': 'application/json'
        }
      });

    } catch (error) {
      return new Response(JSON.stringify({ error: 'Internal Server Error' }), {
        status: 500,
        headers: { ...corsHeaders, 'Content-Type': 'application/json' }
      });
    }
  }
};
```

---

## 🔐 Setting Environment Variables

In Cloudflare dashboard:

**Workers → Your Worker → Settings → Variables → Add**

```
Name: GROQ_API_KEY
Value: <your Groq API key>
```

---

## 🖥️ Frontend

The UI lives in `index.html` and communicates with the Worker:

```js
fetch('https://maghrebi-gpt-backend.maghrebimorteza.workers.dev/', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(requestBody)
});
```

Your API key is **never included in frontend code**.

---

## 🌍 Live Demo

You can test the deployed version on GitHub Pages:

👉 **[https://mortezamaghrebi.github.io/MaghrebiGPT/](https://mortezamaghrebi.github.io/MaghrebiGPT/)**


---

## 📄 License

This project is open-source. Feel free to modify and improve it.

