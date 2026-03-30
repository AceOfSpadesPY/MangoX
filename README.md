🍋 MangoX Economy Plugin (1.12.2+)

MangoX is a custom Minecraft economy plugin built for survival and competitive servers.
It includes a Vault-based economy, a player loan system, and an auction house.

Features

Economy

* Works with Vault
* /pay to send money
* /baltop to see richest players

Loan System

* Players can give loans with interest and time limits
* If not repaid, all money earned goes to the lender
* /loan gui to view loans
* /loantop to see biggest debts

Auction House

* /ah to open GUI
* /ah sell <price> to list items
* Players can buy items directly

Commands

Economy

* /pay <player> <amount>
* /baltop

Loans

* /loan give <player> <amount> <interest max 30> <days min 3>
* /loan gui
* /loantop

Auction House

* /ah
* /ah sell <price>

Current Bugs / Issues

* No data saving yet (resets on restart)
* Basic GUI (not detailed yet)
* Some commands missing error checks
* Auction house may duplicate items if spam-clicked

Requirements

* Spigot 1.12.2+
* Vault
* Economy plugin (like EssentialsX)

Planned Updates

* Save system
* Better GUIs
* Shop system
* Sell system
* Admin controls
