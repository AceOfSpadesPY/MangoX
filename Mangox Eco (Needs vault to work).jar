// MangoX FULL ECONOMY + VAULT + LOANS + GUI + AH
// Spigot 1.12.2+

package com.mangox;

import net.milkbowl.vault.economy.Economy;
import org.bukkit.*;
import org.bukkit.command.*;
import org.bukkit.entity.Player;
import org.bukkit.event.*;
import org.bukkit.event.inventory.InventoryClickEvent;
import org.bukkit.inventory.*;
import org.bukkit.plugin.RegisteredServiceProvider;
import org.bukkit.plugin.java.JavaPlugin;

import java.util.*;

public class MangoX extends JavaPlugin implements Listener {

    private static Economy econ = null;
    public static Map<UUID, List<Loan>> loans = new HashMap<>();
    public static List<AuctionItem> auctionHouse = new ArrayList<>();

    @Override
    public void onEnable() {
        if (!setupEconomy()) {
            getLogger().severe("Vault not found!");
            getServer().getPluginManager().disablePlugin(this);
            return;
        }

        Bukkit.getPluginManager().registerEvents(this, this);

        getCommand("pay").setExecutor(new PayCommand());
        getCommand("baltop").setExecutor(new BaltopCommand());
        getCommand("loan").setExecutor(new LoanCommand());
        getCommand("loantop").setExecutor(new LoanTopCommand());
        getCommand("ah").setExecutor(new AHCommand());

        Bukkit.getScheduler().runTaskTimer(this, this::checkLoans, 20L, 20L * 60);

        getLogger().info("MangoX FULL System Enabled 🍋");
    }

    private boolean setupEconomy() {
        if (getServer().getPluginManager().getPlugin("Vault") == null) return false;
        RegisteredServiceProvider<Economy> rsp = getServer().getServicesManager().getRegistration(Economy.class);
        if (rsp == null) return false;
        econ = rsp.getProvider();
        return econ != null;
    }

    public static Economy getEconomy() {
        return econ;
    }

    // ================= LOAN CHECK =================

    private void checkLoans() {
        long now = System.currentTimeMillis();

        for (UUID borrower : loans.keySet()) {
            Player player = Bukkit.getPlayer(borrower);
            List<Loan> list = loans.get(borrower);

            for (Loan loan : list) {
                if (!loan.paid && now > loan.dueDate) {

                    if (player != null) {
                        double balance = econ.getBalance(player);
                        if (balance > 0) {
                            econ.withdrawPlayer(player, balance);
                            econ.depositPlayer(Bukkit.getOfflinePlayer(loan.lender), balance);
                        }
                    }
                }
            }
        }
    }

    // ================= GUI CLICK =================

    @EventHandler
    public void onClick(InventoryClickEvent e) {
        if (e.getView().getTitle().equals("Loan GUI")) {
            e.setCancelled(true);
        }

        if (e.getView().getTitle().equals("Auction House")) {
            e.setCancelled(true);

            ItemStack clicked = e.getCurrentItem();
            if (clicked == null) return;

            for (AuctionItem item : auctionHouse) {
                if (item.item.isSimilar(clicked)) {
                    Player buyer = (Player) e.getWhoClicked();

                    if (econ.getBalance(buyer) >= item.price) {
                        econ.withdrawPlayer(buyer, item.price);
                        econ.depositPlayer(Bukkit.getOfflinePlayer(item.seller), item.price);
                        buyer.getInventory().addItem(item.item);
                        auctionHouse.remove(item);
                        buyer.sendMessage("Bought item!");
                        break;
                    }
                }
            }
        }
    }
}

// ================= LOAN =================

class Loan {
    UUID lender;
    double amount;
    double interest;
    long dueDate;
    boolean paid;

    public Loan(UUID lender, double amount, double interest, int days) {
        this.lender = lender;
        this.amount = amount;
        this.interest = interest;
        this.dueDate = System.currentTimeMillis() + (days * 86400000L);
        this.paid = false;
    }
}

// ================= AUCTION =================

class AuctionItem {
    UUID seller;
    ItemStack item;
    double price;

    public AuctionItem(UUID seller, ItemStack item, double price) {
        this.seller = seller;
        this.item = item;
        this.price = price;
    }
}

// ================= PAY =================

class PayCommand implements CommandExecutor {
    public boolean onCommand(CommandSender s, Command c, String l, String[] a) {
        if (!(s instanceof Player)) return true;
        Player p = (Player) s;

        Player t = Bukkit.getPlayer(a[0]);
        double amt = Double.parseDouble(a[1]);

        if (MangoX.getEconomy().getBalance(p) < amt) return true;

        MangoX.getEconomy().withdrawPlayer(p, amt);
        MangoX.getEconomy().depositPlayer(t, amt);
        return true;
    }
}

// ================= BALTOP =================

class BaltopCommand implements CommandExecutor {
    public boolean onCommand(CommandSender s, Command c, String l, String[] a) {
        Map<String, Double> map = new HashMap<>();

        for (OfflinePlayer p : Bukkit.getOfflinePlayers()) {
            map.put(p.getName(), MangoX.getEconomy().getBalance(p));
        }

        map.entrySet().stream()
                .sorted((a1, b) -> Double.compare(b.getValue(), a1.getValue()))
                .limit(10)
                .forEach(e -> s.sendMessage(e.getKey() + ": " + e.getValue()));

        return true;
    }
}

// ================= LOAN CMD =================

class LoanCommand implements CommandExecutor {
    public boolean onCommand(CommandSender s, Command c, String l, String[] a) {
        Player p = (Player) s;

        if (a[0].equalsIgnoreCase("give")) {
            Player t = Bukkit.getPlayer(a[1]);
            double amt = Double.parseDouble(a[2]);
            double interest = Double.parseDouble(a[3]);
            int days = Integer.parseInt(a[4]);

            MangoX.getEconomy().withdrawPlayer(p, amt);
            MangoX.getEconomy().depositPlayer(t, amt);

            Loan loan = new Loan(p.getUniqueId(), amt, interest, days);
            MangoX.loans.computeIfAbsent(t.getUniqueId(), k -> new ArrayList<>()).add(loan);
        }

        if (a[0].equalsIgnoreCase("gui")) {
            Inventory inv = Bukkit.createInventory(null, 27, "Loan GUI");

            for (Loan loan : MangoX.loans.getOrDefault(p.getUniqueId(), new ArrayList<>())) {
                ItemStack item = new ItemStack(Material.PAPER);
                inv.addItem(item);
            }

            p.openInventory(inv);
        }

        return true;
    }
}

// ================= LOANTOP =================

class LoanTopCommand implements CommandExecutor {
    public boolean onCommand(CommandSender s, Command c, String l, String[] a) {
        Map<UUID, Double> map = new HashMap<>();

        for (UUID u : MangoX.loans.keySet()) {
            double total = MangoX.loans.get(u).stream().mapToDouble(lo -> lo.amount).sum();
            map.put(u, total);
        }

        map.entrySet().stream()
                .sorted((a1, b) -> Double.compare(b.getValue(), a1.getValue()))
                .limit(10)
                .forEach(e -> s.sendMessage(Bukkit.getOfflinePlayer(e.getKey()).getName() + ": " + e.getValue()));

        return true;
    }
}

// ================= AH =================

class AHCommand implements CommandExecutor {
    public boolean onCommand(CommandSender s, Command c, String l, String[] a) {
        Player p = (Player) s;

        if (a.length == 0) {
            Inventory inv = Bukkit.createInventory(null, 54, "Auction House");
            for (AuctionItem item : MangoX.auctionHouse) {
                inv.addItem(item.item);
            }
            p.openInventory(inv);
        }

        if (a[0].equalsIgnoreCase("sell")) {
            double price = Double.parseDouble(a[1]);
            ItemStack item = p.getInventory().getItemInMainHand();
            MangoX.auctionHouse.add(new AuctionItem(p.getUniqueId(), item, price));
            p.getInventory().setItemInMainHand(null);
        }

        return true;
    }
}

// ================= plugin.yml =================

/*
name: MangoX
main: com.mangox.MangoX
version: 2.0
api-version: 1.12
commands:
  pay:
  baltop:
  loan:
  loantop:
  ah:
*/
