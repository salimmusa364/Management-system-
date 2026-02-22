#include <iostream>
#include <vector>
#include <fstream>
#include <iomanip>

using namespace std;

// -------------------- STUDENT CLASS --------------------
class Student {
private:
    int id;
    string name;

public:
    Student(int id = 0, string name = "") {
        this->id = id;
        this->name = name;
    }

    int getId() const {
        return id;
    }

    string getName() const {
        return name;
    }

    void display() const {
        cout << "ID: " << id << " | Name: " << name << endl;
    }
};

// -------------------- ATTENDANCE SESSION CLASS --------------------
class AttendanceSession {
private:
    string sessionName;
    vector<bool> attendance;  // true = present, false = absent

public:
    AttendanceSession(string name, int studentCount) {
        sessionName = name;
        attendance.resize(studentCount, false);
    }

    string getSessionName() const {
        return sessionName;
    }

    void markAttendance(int index, bool present) {
        if (index >= 0 && index < attendance.size()) {
            attendance[index] = present;
        }
    }

    bool getAttendance(int index) const {
        if (index >= 0 && index < attendance.size()) {
            return attendance[index];
        }
        return false;
    }

    void displaySession(const vector<Student>& students) const {
        cout << "\nSession: " << sessionName << endl;
        for (int i = 0; i < students.size(); i++) {
            cout << students[i].getName() << " - "
                 << (attendance[i] ? "Present" : "Absent") << endl;
        }
    }
};

// -------------------- GLOBAL DATA --------------------
vector<Student> students;
vector<AttendanceSession> sessions;

// -------------------- FUNCTIONS --------------------

void addStudent() {
    int id;
    string name;

    cout << "Enter Student ID: ";
    cin >> id;
    cin.ignore();

    cout << "Enter Student Name: ";
    getline(cin, name);

    students.push_back(Student(id, name));
    cout << "Student added successfully.\n";
}

void viewStudents() {
    if (students.empty()) {
        cout << "No students available.\n";
        return;
    }

    cout << "\nList of Students:\n";
    for (const auto& s : students) {
        s.display();
    }
}

void createSession() {
    if (students.empty()) {
        cout << "Add students first before creating a session.\n";
        return;
    }

    string name;
    cin.ignore();
    cout << "Enter Session Name: ";
    getline(cin, name);

    sessions.push_back(AttendanceSession(name, students.size()));
    cout << "Session created successfully.\n";
}

void markAttendance() {
    if (sessions.empty()) {
        cout << "No sessions available.\n";
        return;
    }

    int sessionIndex;
    cout << "Select Session:\n";
    for (int i = 0; i < sessions.size(); i++) {
        cout << i + 1 << ". " << sessions[i].getSessionName() << endl;
    }

    cout << "Enter session number: ";
    cin >> sessionIndex;

    if (sessionIndex < 1 || sessionIndex > sessions.size()) {
        cout << "Invalid session.\n";
        return;
    }

    AttendanceSession& session = sessions[sessionIndex - 1];

    for (int i = 0; i < students.size(); i++) {
        char choice;
        cout << "Is " << students[i].getName() << " present? (y/n): ";
        cin >> choice;

        session.markAttendance(i, (choice == 'y' || choice == 'Y'));
    }

    cout << "Attendance marked successfully.\n";
}

void generateReport() {
    if (sessions.empty()) {
        cout << "No sessions available.\n";
        return;
    }

    cout << "\nAttendance Summary:\n";

    for (int i = 0; i < students.size(); i++) {
        int presentCount = 0;

        for (const auto& session : sessions) {
            if (session.getAttendance(i))
                presentCount++;
        }

        cout << students[i].getName() << " attended "
             << presentCount << " out of "
             << sessions.size() << " sessions.\n";
    }
}

void saveToFile() {
    ofstream file("attendance.txt");

    if (!file) {
        cout << "Error saving file.\n";
        return;
    }

    file << students.size() << endl;
    for (const auto& s : students) {
        file << s.getId() << "," << s.getName() << endl;
    }

    file << sessions.size() << endl;
    for (const auto& session : sessions) {
        file << session.getSessionName() << endl;
        for (int i = 0; i < students.size(); i++) {
            file << session.getAttendance(i) << " ";
        }
        file << endl;
    }

    file.close();
    cout << "Data saved successfully.\n";
}

void loadFromFile() {
    ifstream file("attendance.txt");

    if (!file) {
        cout << "No saved file found.\n";
        return;
    }

    students.clear();
    sessions.clear();

    int studentCount;
    file >> studentCount;
    file.ignore();

    for (int i = 0; i < studentCount; i++) {
        int id;
        string line, name;

        getline(file, line);
        size_t comma = line.find(",");
        id = stoi(line.substr(0, comma));
        name = line.substr(comma + 1);

        students.push_back(Student(id, name));
    }

    int sessionCount;
    file >> sessionCount;
    file.ignore();

    for (int i = 0; i < sessionCount; i++) {
        string sessionName;
        getline(file, sessionName);

        AttendanceSession session(sessionName, students.size());

        for (int j = 0; j < students.size(); j++) {
            bool present;
            file >> present;
            session.markAttendance(j, present);
        }
        file.ignore();

        sessions.push_back(session);
    }

    file.close();
    cout << "Data loaded successfully.\n";
}

// -------------------- MAIN PROGRAM --------------------

int main() {
    int choice;

    do {
        cout << "\n===== ATTENDANCE MANAGEMENT SYSTEM =====\n";
        cout << "1. Add Student\n";
        cout << "2. View Students\n";
        cout << "3. Create Session\n";
        cout << "4. Mark Attendance\n";
        cout << "5. Generate Report\n";
        cout << "6. Save to File\n";
        cout << "7. Load from File\n";
        cout << "0. Exit\n";
        cout << "Enter choice: ";
        cin >> choice;

        switch (choice) {
            case 1: addStudent(); break;
            case 2: viewStudents(); break;
            case 3: createSession(); break;
            case 4: markAttendance(); break;
            case 5: generateReport(); break;
            case 6: saveToFile(); break;
            case 7: loadFromFile(); break;
            case 0: cout << "Exiting program...\n"; break;
            default: cout << "Invalid choice.\n";
        }

    } while (choice != 0);

    return 0;
}
