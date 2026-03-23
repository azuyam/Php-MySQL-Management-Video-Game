# 🪐 SOLAR_ECONOMY_OS (Space Cargo Trader)

A retro terminal-style resource management game built to demonstrate advanced SQL database concepts, ACID-compliant transactions, and real-time state synchronization using PHP and MySQL.

![HTML5](https://img.shields.io/badge/html5-%23E34F26.svg?style=for-the-badge&logo=html5&logoColor=white) ![CSS3](https://img.shields.io/badge/css3-%231572B6.svg?style=for-the-badge&logo=css3&logoColor=white) ![JavaScript](https://img.shields.io/badge/javascript-%23323330.svg?style=for-the-badge&logo=javascript&logoColor=%23F7DF1E) ![PHP](https://img.shields.io/badge/php-%23777BB4.svg?style=for-the-badge&logo=php&logoColor=white) ![MySQL](https://img.shields.io/badge/mysql-%2300f.svg?style=for-the-badge&logo=mysql&logoColor=white)

## 📖 Overview
**Solar Economy OS** is a UI-centric, browser-based game where the player acts as a corporate logistics pilot. The goal is to keep the populations of Earth and Mars alive by trading essential cargo between them. 

While it functions as a survival game, the project is fundamentally an **interactive database prototype**. Every action the player takes—buying, selling, mining, or traveling—is handled via asynchronous API calls to a PHP backend, executing complex, locked SQL transactions to ensure absolute data integrity.

## 🎮 The Gameplay Loop
* **The Stakes:** Both Earth and Mars constantly consume resources. If either planet reaches 0% Health, the system collapses and the game is over.
* **Mining:** Yields high credits (+50 CR) but damages the local planet orbit (-5 Health) due to orbital debris.
* **Trading:** Buy cheap supplies on one planet, burn resources to travel (-15 Global Health), and sell them at a premium on the other planet.
* **Restoration:** Selling cargo to a planet restores its health. The player must balance greed (hoarding credits) with survival (delivering cargo).

## 🛠️ Core Database Concepts Demonstrated
This project was built to fulfill strict database architecture requirements:

1. **ACID Transactions:** Purchasing and selling cargo utilizes `$pdo->beginTransaction()` and `$pdo->commit()`. If the server drops after deducting credits but before adding the item to the inventory, the entire query is rolled back safely.
2. **Concurrency & Row Locking:** Uses `SELECT ... FOR UPDATE` to lock player rows during transactions. If a user tries to exploit the game by rapid-clicking "Buy", the database processes them sequentially, preventing double-spending or negative balances.
3. **Dynamic Relational Data:** The UI dynamically renders market prices by executing `INNER JOIN` queries between the `items` and `market_prices` associative tables based on the player's current `Location` state.
4. **"Dirty State" Synchronization:** To prevent the database from being overwhelmed by constant micro-updates (like real-time idle decay or rapid mining), the frontend JavaScript batches unrecorded states and safely syncs them to the MySQL database every 5 seconds.

## 🗄️ Database Schema
The database (`space_trader`) consists of 6 core tables:
* `players` - Tracks the session state (Credits, Location).
* `planets` - Tracks the survival status (Health) of Earth and Mars.
* `items` - The read-only dictionary of game entities.
* `market_prices` - An associative entity mapping `ItemID` and `Location` to a dynamic `Price`.
* `inventory` - A linking table tracking what the player currently owns (Hard limit: 5 items).
* `transactions` - A persistent ledger recording every successful `BUY` and `SELL` action.

## 🚀 Installation & Setup (XAMPP)

1. **Install XAMPP:** Download and install XAMPP.
2. **Start Services:** Open the XAMPP Control Panel and start **Apache** and **MySQL**.
3. **Clone the Repository:** 
   Clone or extract this project folder into your XAMPP `htdocs` directory.
   *Path:* `C:/xampp/htdocs/trader/`
4. **Setup the Database:**
   * Open your browser and go to `http://localhost/phpmyadmin/`.
   * Create a new database named `space_trader`.
   * Go to the **SQL** tab and execute the setup queries provided in `database_setup.sql` (or copy them from the project documentation) to build the schema and seed the initial data.
5. **Play the Game:**
   * Open your browser and navigate to `http://localhost/trader/index.html`.

## 📁 File Structure
```text
/trader
│── index.html         # The frontend UI, Web Audio synth, and game loop logic
│── db.php             # PDO Database connection configuration
│── load_game.php      # Fetches total game state (Player, Planets, Inventory, Market)
│── buy_cargo.php      # Handles ACID purchase transactions & capacity limits
│── sell_cargo.php     # Handles selling, transaction logging, and planet healing
│── travel.php         # Updates player location and applies navigation penalties
│── save_mining.php    # Background sync for mining credits, local damage, and idle decay
└── reset_game.php     # TRUNCATE/UPDATE scripts to restart the game loop on Game Over
