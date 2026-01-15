#task_3_sudokusolver
# codealpha_sudokusolver
#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <sstream>
using namespace std;

string hashPassword(const string& password) {
    unsigned long hash = 5381;
    for (char c : password) {
        hash = ((hash << 5) + hash) + c;
    }
    stringstream ss;
    ss << hash;
    return ss.str();
}

bool userExists(const string& username) {
    ifstream infile(username + ".txt");
    return infile.good();
}

void registerUser() {
    string username, password;
    cout << "Enter username: ";
    cin >> username;
    if (userExists(username)) {
        cout << "Username already exists.\n";
        return;
    }
    cout << "Enter password: ";
    cin >> password;

    ofstream outfile(username + ".txt");
    outfile << hashPassword(password) << "\n";
    outfile << 0 << "\n"; // Initial balance
    outfile.close();
    cout << "Registration successful!\n";
}

bool loginUser(string &currentUser) {
    string username, password;
    cout << "Enter username: ";
    cin >> username;
    if (!userExists(username)) {
        cout << "Username does not exist.\n";
        return false;
    }
    cout << "Enter password: ";
    cin >> password;

    ifstream infile(username + ".txt");
    string storedHash;
    infile >> storedHash;
    infile.close();

    if (storedHash == hashPassword(password)) {
        currentUser = username;
        cout << "Login successful! Welcome " << username << ".\n";
        return true;
    } else {
        cout << "Incorrect password.\n";
        return false;
    }
}

double getBalance(const string& username) {
    ifstream infile(username + ".txt");
    string line;
    getline(infile, line); // Skip password hash
    getline(infile, line);
    infile.close();
    return stod(line);
}

void updateBalance(const string& username, double balance) {
    ifstream infile(username + ".txt");
    string passwordHash;
    getline(infile, passwordHash);
    infile.close();

    ofstream outfile(username + ".txt");
    outfile << passwordHash << "\n" << balance << "\n";
    outfile.close();
}

void logTransaction(const string& username, const string& transaction) {
    ofstream outfile(username + "_transactions.txt", ios::app);
    outfile << transaction << "\n";
    outfile.close();
}

void deposit(const string& username) {
    double amount;
    cout << "Enter amount to deposit: ";
    cin >> amount;
    double balance = getBalance(username);
    balance += amount;
    updateBalance(username, balance);
    logTransaction(username, "Deposited: " + to_string(amount));
    cout << "Deposit successful. New Balance: " << balance << "\n";
}

void withdraw(const string& username) {
    double amount;
    cout << "Enter amount to withdraw: ";
    cin >> amount;
    double balance = getBalance(username);
    if (amount > balance) {
        cout << "Insufficient funds.\n";
        return;
    }
    balance -= amount;
    updateBalance(username, balance);
    logTransaction(username, "Withdrew: " + to_string(amount));
    cout << "Withdrawal successful. New Balance: " << balance << "\n";
}

void transfer(const string& fromUser) {
    string toUser;
    double amount;
    cout << "Enter recipient username: ";
    cin >> toUser;
    if (!userExists(toUser)) {
        cout << "Recipient does not exist.\n";
        return;
    }
    cout << "Enter amount to transfer: ";
    cin >> amount;
    double fromBalance = getBalance(fromUser);
    if (amount > fromBalance) {
        cout << "Insufficient funds.\n";
        return;
    }
    double toBalance = getBalance(toUser);
    fromBalance -= amount;
    toBalance += amount;
    updateBalance(fromUser, fromBalance);
    updateBalance(toUser, toBalance);
    logTransaction(fromUser, "Transferred " + to_string(amount) + " to " + toUser);
    logTransaction(toUser, "Received " + to_string(amount) + " from " + fromUser);
    cout << "Transfer successful. New Balance: " << fromBalance << "\n";
}

void viewTransactions(const string& username) {
    ifstream infile(username + "_transactions.txt");
    string line;
    cout << "Recent Transactions:\n";
    while (getline(infile, line)) {
        cout << line << "\n";
    }
    infile.close();
}

void accountMenu(const string& username) {
    int choice;
    do {
        cout << "\n1. Deposit\n2. Withdraw\n3. Transfer\n4. View Balance\n5. View Transactions\n6. Logout\nChoose: ";
        cin >> choice;
        switch(choice) {
            case 1: deposit(username); break;
            case 2: withdraw(username); break;
            case 3: transfer(username); break;
            case 4: cout << "Current Balance: " << getBalance(username) << "\n"; break;
            case 5: viewTransactions(username); break;
            case 6: cout << "Logging out...\n"; break;
            default: cout << "Invalid option.\n";
        }
    } while (choice != 6);
}

int main() {
    int choice;
    string currentUser;
    do {
        cout << "\n1. Register\n2. Login\n3. Exit\nChoose an option: ";
        cin >> choice;
        switch(choice) {
            case 1: registerUser(); break;
            case 2:
                if (loginUser(currentUser))
                    accountMenu(currentUser);
                break;
            case 3: cout << "Exiting...\n"; break;
            default: cout << "Invalid option.\n";
        }
    } while(choice != 3);
    return 0;
}
