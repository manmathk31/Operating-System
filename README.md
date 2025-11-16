
**Name:** Manmath Maroti Kornule  
**PRN:** 202401110045

---

# 💻 Page Replacement Algorithm Simulator

This is a C++ program that simulates three fundamental page replacement algorithms used in operating systems for managing memory:

* **FIFO (First-In, First-Out):** Replaces the page that has been in memory the longest.
* **LRU (Least Recently Used):** Replaces the page that has not been accessed for the longest time.
* **Optimal (OPT):** Replaces the page that will not be used for the longest period in the future (the ideal, non-real-time algorithm).

The program is menu-driven, allowing you to enter a custom page reference string and the number of available memory frames. It then provides a detailed, step-by-step simulation for each algorithm, showing page hits, faults, and the state of the frames at each step.

---

## 🚀 The Code

Below is the complete C++ source code for the simulator.

```cpp

#include <iostream>
#include <vector>
#include <sstream>
#include <iomanip>
#include <limits>
using namespace std;

// ------------------------------------------------------------
// Utility Functions
// ------------------------------------------------------------
void printLine() {
    cout << "------------------------------------------------------------\n";
}

// Displays current frame state
void printFrames(const vector<int>& frames) {
    cout << "| ";
    for (int f : frames) {
        if (f == -1) cout << "- ";
        else cout << f << " ";
    }
    cout << " ";
}

// ------------------------------------------------------------
// FIFO Algorithm
// ------------------------------------------------------------
int simulateFIFO(const vector<int>& refs, int framesCount) {
    printLine();
    cout << "Starting FIFO Simulation...\n";
    printLine();

    cout << "Reference String: ";
    for (int r : refs) cout << r << " ";
    cout << "\nTotal Frames: " << framesCount << "\n\n";

    printLine();
    cout << "Step-by-step Page Frame Updates:\n\n";
    cout << "Ref   Frame Status                 Page Fault?\n";
    printLine();

    vector<int> frames(framesCount, -1);
    int pointer = 0, faults = 0;

    for (int r : refs) {
        bool hit = false;

        for (int f : frames) {
            if (f == r) {
                hit = true;
                break;
            }
        }

        cout << left << setw(5) << r;

        if (!hit) {
            cout << " ";
            frames[pointer] = r;
            pointer = (pointer + 1) % framesCount;
            faults++;

            printFrames(frames);
            cout << "    YES (Page Loaded)\n";
        } else {
            printFrames(frames);
            cout << "    NO  (Already in memory)\n";
        }
    }

    printLine();
    cout << "Simulation Results (FIFO)\n";
    printLine();
    cout << "Total Page Faults: " << faults << "\n";
    cout << "Total References : " << refs.size() << "\n";
    cout << "Fault Rate       : " << fixed << setprecision(2)
         << (faults * 100.0 / refs.size()) << "%\n\n";

    cout << "FIFO Explanation:\n";
    cout << "FIFO replaces the oldest loaded page first, regardless of\n";
    cout << "how frequently or recently it is used. This often leads to\n";
    cout << "unnecessary faults and may suffer from Belady's anomaly.\n";

    printLine();
    cout << "End of Simulation.\n";
    cout << "You may run another algorithm from the menu.\n";
    printLine();

    return faults;
}

// ------------------------------------------------------------
// LRU Algorithm
// ------------------------------------------------------------
int simulateLRU(const vector<int>& refs, int framesCount) {
    printLine();
    cout << "Starting LRU Simulation...\n";
    printLine();

    cout << "Reference String: ";
    for (int r : refs) cout << r << " ";
    cout << "\nTotal Frames: " << framesCount << "\n\n";

    printLine();
    cout << "Step-by-step Page Frame Updates:\n\n";
    cout << "Ref   Frame Status                 Page Fault?\n";
    printLine();

    vector<int> frames(framesCount, -1);
    vector<int> lastUsed(framesCount, -1);
    int faults = 0, time = 0;

    for (int r : refs) {
        time++;
        bool hit = false;
        int hitIndex = -1;

        // Check hit
        for (int i = 0; i < framesCount; i++) {
            if (frames[i] == r) {
                hit = true;
                hitIndex = i;
                break;
            }
        }

        cout << left << setw(5) << r;

        if (hit) {
            lastUsed[hitIndex] = time;
            printFrames(frames);
            cout << "    NO  (Already in memory)\n";
        } else {
            faults++;

            // Find empty frame
            int replaceIndex = -1;
            for (int i = 0; i < framesCount; i++) {
                if (frames[i] == -1) {
                    replaceIndex = i;
                    break;
                }
            }

            // If no empty frame → replace LRU
            if (replaceIndex == -1) {
                int lruTime = numeric_limits<int>::max();
                for (int i = 0; i < framesCount; i++) {
                    if (lastUsed[i] < lruTime) {
                        lruTime = lastUsed[i];
                        replaceIndex = i;
                    }
                }

                cout << " ";
                int replaced = frames[replaceIndex];
                frames[replaceIndex] = r;
                lastUsed[replaceIndex] = time;

                printFrames(frames);
                cout << "    YES (Replaced LRU: " << replaced << ")\n";
            } else {
                frames[replaceIndex] = r;
                lastUsed[replaceIndex] = time;
                printFrames(frames);
                cout << "    YES (Loaded into empty frame)\n";
            }
        }
    }

    printLine();
    cout << "Simulation Results (LRU)\n";
    printLine();
    cout << "Total Page Faults: " << faults << "\n";
    cout << "Total References : " << refs.size() << "\n";
    cout << "Fault Rate       : " << fixed << setprecision(2)
         << (faults * 100.0 / refs.size()) << "%\n\n";

    cout << "LRU Explanation:\n";
    cout << "LRU replaces the page that has not been used for the longest\n";
    cout << "time. It performs well with workloads showing locality but\n";
    cout << "may fault more when memory access patterns change suddenly.\n";

    printLine();
    cout << "End of Simulation.\n";
    cout << "You may run another algorithm from the menu.\n";
    printLine();

    return faults;
}

// ------------------------------------------------------------
// Optimal Algorithm
// ------------------------------------------------------------
int simulateOptimal(const vector<int>& refs, int framesCount) {
    printLine();
    cout << "Starting Optimal Simulation...\n";
    printLine();

    cout << "Reference String: ";
    for (int r : refs) cout << r << " ";
    cout << "\nTotal Frames: " << framesCount << "\n\n";

    printLine();
    cout << "Step-by-step Page Frame Updates:\n\n";
    cout << "Ref   Frame Status                 Page Fault?\n";
    printLine();

    vector<int> frames(framesCount, -1);
    int faults = 0;

    for (int i = 0; i < refs.size(); i++) {
        int r = refs[i];
        bool hit = false;

        // Check if already present
        for (int f : frames) {
            if (f == r) { hit = true; break; }
        }

        cout << left << setw(5) << r;

        if (hit) {
            printFrames(frames);
            cout << "    NO  (Already in memory)\n";
            continue;
        }

        faults++;

        // Find empty frame
        int emptyIndex = -1;
        for (int j = 0; j < framesCount; j++) {
            if (frames[j] == -1) {
                emptyIndex = j;
                break;
            }
        }

        if (emptyIndex != -1) {
            frames[emptyIndex] = r;
            printFrames(frames);
            cout << "    YES (Loaded into empty frame)\n";
        } else {
            // Find optimal replacement
            int farthest = -1, replaceIndex = -1;

            for (int j = 0; j < framesCount; j++) {
                int nextUse = -1;

                for (int k = i + 1; k < refs.size(); k++) {
                    if (frames[j] == refs[k]) {
                        nextUse = k;
                        break;
                    }
                }

                if (nextUse == -1) { // never used again
                    replaceIndex = j;
                    break;
                }

                if (nextUse > farthest) {
                    farthest = nextUse;
                    replaceIndex = j;
                }
            }

            int replaced = frames[replaceIndex];
            frames[replaceIndex] = r;

            printFrames(frames);
            cout << "    YES (Replaced Optimal: " << replaced << ")\n";
        }
    }

    printLine();
    cout << "Simulation Results (Optimal)\n";
    printLine();
    cout << "Total Page Faults: " << faults << "\n";
    cout << "Total References : " << refs.size() << "\n";
    cout << "Fault Rate       : " << fixed << setprecision(2)
         << (faults * 100.0 / refs.size()) << "%\n\n";

    cout << "Optimal Explanation:\n";
    cout << "Optimal replacement uses future knowledge to replace the page\n";
    cout << "that will not be used for the longest time. It gives the\n";
    cout << "minimum possible number of page faults but cannot be\n";
    cout << "implemented in real systems. It is used only for comparison.\n";

    printLine();
    cout << "End of Simulation.\n";
    cout << "You may run another algorithm from the menu.\n";
    printLine();

    return faults;
}

// ------------------------------------------------------------
// Main Program Menu
// ------------------------------------------------------------
int main() {
    while (true) {
        printLine();
        cout << "              PAGE REPLACEMENT ALGORITHM SIMULATOR\n";
        printLine();
        cout << "This program allows you to experiment with different\n";
        cout << "page replacement strategies. You can provide your own\n";
        cout << "reference string and frame size, and observe how each\n";
        cout << "algorithm manages memory.\n\n";

        cout << "Please choose an algorithm to simulate:\n";
        cout << "1. FIFO (First-In First-Out)\n";
        cout << "2. LRU  (Least Recently Used)\n";
        cout << "3. Optimal Page Replacement\n";
        cout << "4. Exit\n\n";

        cout << "Enter your choice: ";
        int choice;
        cin >> choice;

        if (choice == 4) break;

        cin.ignore(numeric_limits<streamsize>::max(), '\n');
        printLine();

        cout << "Enter the reference string (space-separated): ";
        string input;
        getline(cin, input);

        stringstream ss(input);
        vector<int> refs;
        int num;
        while (ss >> num) refs.push_back(num);

        cout << "Enter the number of frames: ";
        int frames;
        cin >> frames;

        printLine();

        if (choice == 1) simulateFIFO(refs, frames);
        else if (choice == 2) simulateLRU(refs, frames);
        else if (choice == 3) simulateOptimal(refs, frames);
        else cout << "Invalid option.\n";
    }

    cout << "\nThank you for using the simulator.\n";
    return 0;
}


```
