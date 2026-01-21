# bank-management-system
This Java-based Bank Management System connects to a MySQL database to perform basic banking operations. It allows users to create new accounts, deposit funds, withdraw money, and check account balances. The system uses JDBC for database interactions and features a simple command-line interface


    package anudeep;


import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.Scanner;

public class BankApp {

    static final String URL =
        "jdbc:mysql://localhost:3306/bankdb?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC";
    static final String USER = "root";
    static final String PASS = "";   // XAMPP default

    static Connection getConnection() throws Exception {
        Class.forName("com.mysql.cj.jdbc.Driver");
        return DriverManager.getConnection(URL, USER, PASS);
    }

    static void createAccount(Scanner sc) {
        try (Connection con = getConnection()) {
            System.out.print("Enter Name: ");
            sc.nextLine(); // clear buffer
            String name = sc.nextLine();
            System.out.print("Enter Initial Balance: ");
            double bal = sc.nextDouble();

            PreparedStatement ps = con.prepareStatement(
                "INSERT INTO accounts(name, balance) VALUES (?, ?)");
            ps.setString(1, name);
            ps.setDouble(2, bal);
            ps.executeUpdate();

            System.out.println("✅ Account Created Successfully!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void deposit(Scanner sc) {
        try (Connection con = getConnection()) {
            System.out.print("Enter Account No: ");
            int acc = sc.nextInt();
            System.out.print("Enter Amount: ");
            double amt = sc.nextDouble();

            PreparedStatement ps = con.prepareStatement(
                "UPDATE accounts SET balance = balance + ? WHERE acc_no = ?");
            ps.setDouble(1, amt);
            ps.setInt(2, acc);

            int rows = ps.executeUpdate();
            if (rows > 0)
                System.out.println("✅ Deposit Successful!");
            else
                System.out.println("❌ Account Not Found!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void withdraw(Scanner sc) {
        try (Connection con = getConnection()) {
            System.out.print("Enter Account No: ");
            int acc = sc.nextInt();
            System.out.print("Enter Amount: ");
            double amt = sc.nextDouble();

            PreparedStatement ps1 = con.prepareStatement(
                "SELECT balance FROM accounts WHERE acc_no = ?");
            ps1.setInt(1, acc);
            ResultSet rs = ps1.executeQuery();

            if (rs.next()) {
                double bal = rs.getDouble(1);
                if (bal >= amt) {
                    PreparedStatement ps2 = con.prepareStatement(
                        "UPDATE accounts SET balance = balance - ? WHERE acc_no = ?");
                    ps2.setDouble(1, amt);
                    ps2.setInt(2, acc);
                    ps2.executeUpdate();
                    System.out.println("✅ Withdrawal Successful!");
                } else {
                    System.out.println("❌ Insufficient Balance!");
                }
            } else {
                System.out.println("❌ Account Not Found!");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void checkBalance(Scanner sc) {
        try (Connection con = getConnection()) {
            System.out.print("Enter Account No: ");
            int acc = sc.nextInt();

            PreparedStatement ps = con.prepareStatement(
                "SELECT name, balance FROM accounts WHERE acc_no = ?");
            ps.setInt(1, acc);
            ResultSet rs = ps.executeQuery();

            if (rs.next()) {
                System.out.println("👤 Name: " + rs.getString(1));
                System.out.println("💰 Balance: ₹" + rs.getDouble(2));
            } else {
                System.out.println("❌ Account Not Found!");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        while (true) {
            System.out.println("\n--- BANK MANAGEMENT SYSTEM ---");
            System.out.println("1. Create Account");
            System.out.println("2. Deposit");
            System.out.println("3. Withdraw");
            System.out.println("4. Check Balance");
            System.out.println("5. Exit");
            System.out.print("Enter choice: ");

            int ch = sc.nextInt();

            switch (ch) {
                case 1: createAccount(sc); break;
                case 2: deposit(sc); break;
                case 3: withdraw(sc); break;
                case 4: checkBalance(sc); break;
                case 5:
                    System.out.println("🙏 Thank you!");
                    sc.close();
                    System.exit(0);
            }
        }
    }
}
