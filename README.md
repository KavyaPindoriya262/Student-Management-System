# Student-Management-System
#include <iostream>
#include <string>
#include <iomanip>
using namespace std;

struct Student {
    int student_id;
    string name;
    float marks;
    string course;
    Student* next;
};

// Global head pointer for the main linked list
Student* head = nullptr;

// Stack node for undo (store deleted records)
struct UndoNode {
    Student data;
    UndoNode* next;
};
UndoNode* undo_stack = nullptr;

// Check if student_id already exists
bool exists(int id) {
    Student* tmp = head;
    while (tmp != nullptr) {
        if (tmp->student_id == id) return true;
        tmp = tmp->next;
    }
    return false;
}

// Add a student record into the linked list
void add_student() {
    int id;
    string name, course;
    float marks;

    cout << "Enter Student ID: ";
    cin >> id;

    if (exists(id)) {
        cout << "Student with ID " << id << " already exists!";
        return;
    }

    cout << "Enter Name: ";
    cin.ignore();
    getline(cin, name);
    cout << "Enter Course: ";
    getline(cin, course);
    cout << "Enter Marks: ";
    cin >> marks;

    Student* new_student = new Student();
    new_student->student_id = id;
    new_student->name = name;
    new_student->marks = marks;
    new_student->course = course;
    new_student->next = nullptr;

    // Insert at the beginning (simple insertion)
    new_student->next = head;
    head = new_student;

    cout << "Record inserted successfully.";
}

// Search student by ID
void search_student() {
    int id;
    cout << "Enter Student ID to search: ";
    cin >> id;

    Student* tmp = head;
    while (tmp != nullptr) {
        if (tmp->student_id == id) {
            cout << "--- Student Found ---";
            cout << "ID     : " << tmp->student_id << " ";
            cout << "Name   : " << tmp->name << "";
            cout << "Course : " << tmp->course << "";
            cout << "Marks  : " << fixed << setprecision(2) << tmp->marks << "";
            return;
        }
        tmp = tmp->next;
    }

    cout << "No record found with ID " << id << ".";
}

// Push onto undo stack (store deleted record)
void push_undo(Student data) {
    UndoNode* node = new UndoNode();
    node->data = data;
    node->next = undo_stack;
    undo_stack = node;
}

// Pop from undo stack and return the data
bool pop_undo(Student& data) {
    if (undo_stack == nullptr) return false;

    UndoNode* top = undo_stack;
    data = top->data;
    undo_stack = undo_stack->next;
    delete top;
    return true;
}

// Delete a student by ID (with undo)
void delete_student() {
    int id;
    cout << "Enter Student ID to delete: ";
    cin >> id;

    Student* current = head;
    Student* prev = nullptr;

    while (current != nullptr) {
        if (current->student_id == id) {
            // Push deleted record to undo stack
            push_undo(*current);

            if (prev == nullptr) {
                // Delete head
                head = head->next;
            } else {
                prev->next = current->next;
            }

            delete current;
            cout << "Record with ID " << id << " deleted.";
            return;
        }
        prev = current;
        current = current->next;
    }

    cout << "No record found with ID " << id << ".";
}

// Undo last delete (re‑insert from undo stack)
void undo_last_delete() {
    Student data;
    if (!pop_undo(data)) {
        cout << "No delete operation to undo.";
        return;
    }

    // Just re‑insert at the beginning (like add)
    Student* new_student = new Student();
    new_student->student_id = data.student_id;
    new_student->name = data.name;
    new_student->marks = data.marks;
    new_student->course = data.course;
    new_student->next = head;
    head = new_student;

    cout << "Last deleted record (ID " << data.student_id << ") restored.";
}

// Display all students
void display_all() {
    if (head == nullptr) {
        cout << "No records available.";
        return;
    }

    cout << "" << setw(5) << "ID"
         << setw(20) << "Name"
         << setw(15) << "Course"
         << setw(10) << "Marks";
    cout << string(50, '-') << "";

    Student* tmp = head;
    while (tmp != nullptr) {
        cout << setw(5) << tmp->student_id
             << setw(20) << tmp->name
             << setw(15) << tmp->course
             << setw(10) << fixed << setprecision(2) << tmp->marks << "";
        tmp = tmp->next;
    }
}

// Sort students by marks (descending, using selection‑sort style on the list)
void sort_students_by_marks() {
    if (head == nullptr || head->next == nullptr) {
        cout << "No need to sort (0 or 1 record).";
        return;
    }

    Student* i = head;
    while (i != nullptr) {
        Student* max_node = i;
        Student* j = i->next;

        // Find maximum marks node from i onwards
        while (j != nullptr) {
            if (j->marks > max_node->marks) {
                max_node = j;
            }
            j = j->next;
        }

        // Swap data (not pointers, for simplicity)
        if (max_node != i) {
            swap(i->student_id, max_node->student_id);
            swap(i->name, max_node->name);
            swap(i->marks, max_node->marks);
            swap(i->course, max_node->course);
        }

        i = i->next;
    }

    cout << "Students sorted by marks (descending).";
}

// Show main menu
void show_menu() {
    cout << "\n\t--- STUDENT RECORD MANAGEMENT SYSTEM ---\n";
    cout << "1. Add Student\n";
    cout << "2. Delete Student\n";
    cout << "3. Search Student by ID\n";
    cout << "4. Display All Students\n";
    cout << "5. Sort Students by Marks (Descending)\n";
    cout << "6. Undo Last Delete\n";
    cout << "7. Exit\n";
    cout << "Enter your choice: ";
}
// Main function
int main() {
    int choice;

    while (true) {
        show_menu();
        cin >> choice;

        switch (choice) {
            case 1:
                add_student();
                break;
            case 2:
                delete_student();
                break;
            case 3:
                search_student();
                break;
            case 4:
                display_all();
                break;
            case 5:
                sort_students_by_marks();
                break;
            case 6:
                undo_last_delete();
                break;
            case 7:
                cout << "Exiting program.";
                return 0;
            default:
                cout << "Invalid choice. Try again.";
        }
    }

    return 0;
}
