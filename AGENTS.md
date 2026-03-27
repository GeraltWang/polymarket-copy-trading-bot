# AGENTS.md — Polymarket Copy Trading Bot

## Project Overview

A TypeScript-based automated copy trading bot for [Polymarket](https://polymarket.com). It monitors trader activity on Polymarket via the CLOB API, calculates proportional position sizes, and executes matching orders in real-time. Trade history is persisted in MongoDB.

## Tech Stack

- **Language**: TypeScript (strict mode, ES2016 target, CommonJS modules)
- **Runtime**: Node.js v18+
- **Blockchain**: Polygon (ethers v5), Polymarket CLOB Client (`@polymarket/clob-client`)
- **Database**: MongoDB via Mongoose
- **HTTP**: Axios
- **Build**: `tsc` (source in `src/`, output in `dist/`)
- **Test**: Jest + ts-jest
- **Lint/Format**: ESLint (flat config) + Prettier (4-space indent, single quotes, semicolons, trailing comma es5)

## Project Structure

```
src/
├── index.ts              # Entry point — connects DB, initializes CLOB client, starts monitor & executor
├── config/
│   ├── env.ts            # Environment variable parsing & validation (exports ENV object)
│   ├── db.ts             # MongoDB connection (Mongoose)
│   └── copyStrategy.ts   # Copy strategy types (PERCENTAGE/FIXED/ADAPTIVE), order size calculation, tiered multipliers
├── interfaces/
│   └── User.ts           # TypeScript interfaces for user/trade data
├── models/
│   └── userHistory.ts    # Mongoose schema for trade history
├── services/
│   ├── tradeMonitor.ts   # Polls Polymarket API for new trader positions
│   ├── tradeExecutor.ts  # Executes copy trades via CLOB client
│   └── createClobClient.ts
├── utils/
│   ├── createClobClient.ts  # CLOB client factory
│   ├── fetchData.ts         # HTTP data fetching with retry/timeout
│   ├── getMyBalance.ts      # USDC balance queries
│   ├── postOrder.ts         # Order submission logic
│   ├── healthCheck.ts       # System health verification
│   ├── logger.ts            # Colored console logging utility
│   └── spinner.ts           # CLI spinner
└── scripts/              # Standalone CLI tools (run via `npm run <script>`)
    ├── healthCheck.ts     # Verify configuration & connectivity
    ├── setup.ts           # Interactive setup wizard
    ├── manualSell.ts      # Manual position selling
    ├── findBestTraders.ts # Discover profitable traders
    ├── simulateProfitability.ts  # Backtest copy trading strategies
    └── ...                # 30+ utility scripts
```

## Key Commands

```bash
npm run dev              # Run bot in development mode (ts-node)
npm run build            # Compile TypeScript to dist/
npm start                # Run compiled bot (node dist/index.js)
npm run health-check     # Verify configuration & connectivity
npm run lint             # Run ESLint
npm run lint:fix         # Auto-fix ESLint issues
npm run format           # Format code with Prettier
```

## Code Conventions

- **Formatting**: Prettier with 4-space indentation, single quotes, semicolons, trailing commas (es5), 100 char print width.
- **Imports**: Use named imports. `dotenv` is configured in `src/config/env.ts` only.
- **Configuration**: All env vars are parsed and validated in `src/config/env.ts` and exported as the `ENV` object. Never read `process.env` directly elsewhere.
- **Logging**: Use the `Logger` utility (`src/utils/logger.ts`) — do NOT use `console.log` in services/utils (config files may use `console.error` for validation messages).
- **Error handling**: The bot has graceful shutdown (SIGTERM/SIGINT). Unhandled rejections are logged but don't crash; uncaught exceptions trigger shutdown.
- **Module system**: CommonJS (`"type": "commonjs"` in package.json). Use `import/export`, not `require`.

## Important Notes

- **Security**: Never log or expose `PRIVATE_KEY`, `MONGO_URI`, or any secrets. The `.env` file is gitignored.
- **Blockchain context**: The bot operates on Polygon mainnet. USDC contract address is fixed. Orders go through Polymarket's CLOB (Central Limit Order Book) API.
- **Copy strategies**: Three strategies exist — `PERCENTAGE`, `FIXED`, `ADAPTIVE`. Tiered multipliers can be layered on top. All sizing logic lives in `src/config/copyStrategy.ts`.
- **Scripts**: The `src/scripts/` directory contains standalone CLI tools. They import from config/utils but are not part of the main bot loop. Each is mapped to an npm script in `package.json`.
