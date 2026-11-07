# Kairo: 3D cookie website with an AI chatbot

A functional web application. The website shows a 3D black sandwich-cookie brand ("Kairo"), and a chatbot named **Kai** answers questions using a real AI model.

## What it does
- **3D flavor carousel**: 6 flavors, drag/click/arrow keys, the whole page changes colour per flavor
- **Scroll story**: Twist, Lick the cream, Dunk, Crumble (the 3D cookie animates as you scroll)
- **Flavor wall** and **Build-a-Box** (pick 3 packs, price $8.99, basket counter)
- **AI chatbot (Kai)**: recommends flavors, explains prices and the box deal, and shows "Add to box" buttons inside the chat
- **Offline demo mode**: if there is no AI key (or the AI fails), a built-in rule-based bot still answers, so the demo never breaks

## Run it in VS Code (3 steps)
1. Install **Node.js 18 or newer** (https://nodejs.org). Check with `node --version`.
2. Get a free AI key: open https://aistudio.google.com/app/apikey, sign in, click **Create API key**.
   Then copy `.env.example` to a new file called `.env` and paste the key:
   ```
   GEMINI_API_KEY=your_key_here
   ```
3. Open this folder in VS Code, open the terminal (Ctrl + `) and run:
   ```
   npm start
   ```
   Open **http://localhost:3000**. The terminal shows `AI provider: gemini (live AI replies)`.

No `npm install` is needed because the server uses only built-in Node.js modules.

Without a key the site still works, and the chat badge says **Offline demo**.

## Project files
| File | Purpose |
|---|---|
| `server.js` | Node.js server: serves the website, receives chat messages, calls the AI, returns the reply |
| `public/index.html` | The whole website: 3D engine (WebGL2), animations, shop, and the chat widget |
| `.env` | Your secret API key (never share or upload this) |
| `.env.example` | Template for `.env` |
| `package.json` | Project info and the `npm start` command |

## How the chatbot works
```
Browser (chat box)  --POST /api/chat-->  Node server  --API call + key-->  AI model (Gemini)
        ^                                    |                                  |
        |<------------- reply ---------------+<---------- answer text ----------+
        v
 shows the message and turns flavor names into "Add to box" buttons
```
1. You type a message. The page sends the last 10 messages to `/api/chat`.
2. `server.js` cleans the input (max 500 characters per message, only `user`/`assistant` roles), applies a rate limit (20 questions per minute), and adds a **system prompt** that tells the AI it is Kai and gives it the flavor and price facts.
3. The server calls the AI provider with the secret key and returns the text.
4. The page shows the reply. If a flavor name appears in it, an **Add to box** button is created.
5. If the server or AI is unavailable, `localBot()` in the page answers using keyword rules.

**Why a server?** An API key placed in browser code can be stolen by anyone who opens the page. Keeping it in `.env` on the server keeps it private.

**Other AI providers:** put `ANTHROPIC_API_KEY` or `OPENAI_API_KEY` in `.env` instead. You can also set `AI_PROVIDER` and the model name (see `.env.example`).

## How the 3D works (short version)
- No 3D library: the cookies are drawn with **WebGL2** and a hand-written shader.
- Each cookie is built in code from three parts: a black wafer (a lathe shape), a lumpy cream layer, and a second wafer. The Kairo logo image is wrapped onto the wafer top and used as a bump map so it looks embossed.
- The page scroll position drives the twist, lift, dunk and crumble animation.

## Demo script (3 minutes)
1. Show the site: drag the carousel, notice the colours change with each flavor.
2. Scroll the story: twist, cream, dunk, crumble.
3. Build a box on the wall: pick 3 flavors, show the price and the basket.
4. Open **Ask Kai**: ask "I like minty and fruity, what should I try?" and click the Add button.
5. Show VS Code: `server.js` (the `/api/chat` route and the system prompt) and the terminal log lines like `[chat] gemini 640ms`.
6. Stop the key (rename `.env`), restart, and show **Offline demo** mode still answers.

## Troubleshooting
| Problem | Fix |
|---|---|
| Badge says "Offline demo" | `.env` is missing, has no key, or the server was not restarted after editing it |
| "The AI service said: API key not valid" | Re-copy the key into `.env` with no spaces or quotes, then restart |
| "Model not found" | Set `GEMINI_MODEL` in `.env` to a model shown in Google AI Studio |
| `node` is not recognised | Install Node.js and reopen VS Code |
| Port in use | Add `PORT=3001` to `.env` |
| Page loads but chat only says "Offline demo" when opened as a file | Open it through `http://localhost:3000`, not by double-clicking `index.html` |
