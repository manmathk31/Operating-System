
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

#include <bits/stdc++.h>
using namespace std;

/* ---------------------------------------------------------
   PAGE REPLACEMENT SIMULATOR (Menu Driven)

   - Presents detailed, instructional, user-friendly output.
   - Shows each step clearly: reference number, hit/fault,
     eviction, frame status.
   - No unnecessary symbols or emojis.
   --------------------------------------------------------- */

void printFrames(const vector<int>& frames) {
    cout << "Frames: [ ";
    for (int f : frames) {
        if (f == -1) cout << "_ ";
        else cout << f << " ";
    }
    cout << "]";
}

/* ======================== FIFO ======================== */
int simulateFIFO(const vector<int>& refs, int frames_count) {
    cout << "\n------------------------------------------------------------\n";
    cout << "                     FIFO PAGE REPLACEMENT                  \n";
    cout << "------------------------------------------------------------\n";
    cout << "Explanation: The page that entered first is removed first.\n\n";

    vector<int> frames(frames_count, -1);
    queue<int> q;
    unordered_set<int> present;
    int faults = 0;

    for (int i = 0; i < refs.size(); i++) {
        int page = refs[i];

        cout << "Reference " << i + 1 << " -> Page " << page << "\n";

        if (present.count(page)) {
            cout << "Result: HIT (page already in memory)\n";
        } else {
            faults++;
            cout << "Result: PAGE FAULT\n";

            if (present.size() < frames_count) {
                for (int j = 0; j < frames_count; j++) {
                    if (frames[j] == -1) {
                        frames[j] = page;
                        break;
                    }
                }
                present.insert(page);
                q.push(page);
                cout << "Action: Page loaded into an empty frame.\n";
            } else {
                int victim = q.front();
                q.pop();
                cout << "Action: Evicted page " << victim << " (oldest page in memory).\n";

                for (int j = 0; j < frames_count; j++) {
                    if (frames[j] == victim) {
                        frames[j] = page;
                        break;
                    }
                }

                present.erase(victim);
                present.insert(page);
                q.push(page);
            }
        }

        printFrames(frames);
        cout << "\n\n";
    }

    cout << "Total FIFO Page Faults = " << faults << "\n";
    return faults;
}

/* ======================== LRU ======================== */
int simulateLRU(const vector<int>& refs, int frames_count) {
    cout << "\n------------------------------------------------------------\n";
    cout << "                 LRU PAGE REPLACEMENT                       \n";
    cout << "------------------------------------------------------------\n";
    cout << "Explanation: Removes the page that has not been used recently.\n\n";

    vector<int> frames(frames_count, -1);
    unordered_map<int, int> lastUsed;

    int faults = 0;

    for (int time = 0; time < refs.size(); time++) {
        int page = refs[time];

        cout << "Reference " << time + 1 << " -> Page " << page << "\n";

        bool hit = false;
        for (int f : frames) if (f == page) hit = true;

        if (hit) {
            cout << "Result: HIT (page used recently)\n";
            lastUsed[page] = time;
        } else {
            faults++;
            cout << "Result: PAGE FAULT\n";

            bool placed = false;
            for (int j = 0; j < frames_count; j++) {
                if (frames[j] == -1) {
                    frames[j] = page;
                    lastUsed[page] = time;
                    placed = true;
                    cout << "Action: Page placed in an empty frame.\n";
                    break;
                }
            }

            if (!placed) {
                int lruPage = -1, lruTime = INT_MAX;

                for (int f : frames) {
                    int t = lastUsed[f];
                    if (t < lruTime) {
                        lruTime = t;
                        lruPage = f;
                    }
                }

                cout << "Action: Evicted page " << lruPage 
                     << " (least recently used).\n";

                for (int j = 0; j < frames_count; j++) {
                    if (frames[j] == lruPage) {
                        frames[j] = page;
                    }
                }

                lastUsed.erase(lruPage);
                lastUsed[page] = time;
            }
        }

        printFrames(frames);
        cout << "\n\n";
    }

    cout << "Total LRU Page Faults = " << faults << "\n";
    return faults;
}

/* ====================== OPTIMAL ====================== */
int simulateOptimal(const vector<int>& refs, int frames_count) {
    cout << "\n------------------------------------------------------------\n";
    cout << "                     OPTIMAL PAGE REPLACEMENT               \n";
    cout << "------------------------------------------------------------\n";
    cout << "Explanation: Replaces the page that will not be needed for the longest time.\n\n";

    vector<int> frames(frames_count, -1);
    int faults = 0;

    for (int i = 0; i < refs.size(); i++) {
        int page = refs[i];

        cout << "Reference " << i + 1 << " -> Page " << page << "\n";

        bool hit = false;
        for (int f : frames) if (f == page) hit = true;

        if (hit) {
            cout << "Result: HIT (page already in memory)\n";
        } else {
            faults++;
            cout << "Result: PAGE FAULT\n";

            bool placed = false;
            for (int j = 0; j < frames_count; j++) {
                if (frames[j] == -1) {
                    frames[j] = page;
                    placed = true;
                    cout << "Action: Page placed in an empty frame.\n";
                    break;
                }
            }

            if (!placed) {
                int victimIndex = -1;
                int farthestUse = -1;

                for (int j = 0; j < frames_count; j++) {
                    int current = frames[j];
                    int nextUse = INT_MAX;

                    for (int k = i + 1; k < refs.size(); k++) {
                        if (refs[k] == current) {
                            nextUse = k;
                            break;
                        }
                    }

                    if (nextUse > farthestUse) {
                        farthestUse = nextUse;
                        victimIndex = j;
                    }
                }

                int victim = frames[victimIndex];
                cout << "Action: Evicted page " << victim 
                     << " (used farthest in the future).\n";

                frames[victimIndex] = page;
            }
        }

        printFrames(frames);
        cout << "\n\n";
    }

    cout << "Total OPTIMAL Page Faults = " << faults << "\n";
    return faults;
}

/* ========================== MAIN ========================= */
int main() {
    cout << "============================================================\n";
    cout << "            PAGE REPLACEMENT ALGORITHM SIMULATOR            \n";
    cout << "============================================================\n";
    cout << "This program demonstrates how memory pages are loaded and\n";
    cout << "replaced under different page replacement strategies.\n";
    cout << "You can run FIFO, LRU, or OPTIMAL multiple times using menu.\n\n";

    int n;
    cout << "Enter the length of the reference string: ";
    cin >> n;

    vector<int> refs(n);
    cout << "Enter " << n << " page references: ";
    for (int i = 0; i < n; i++) cin >> refs[i];

    int frames;
    cout << "Enter number of frames available in memory: ";
    cin >> frames;

    int choice;

    do {
        cout << "\n---------------------- MENU ----------------------\n";
        cout << "1. Simulate FIFO\n";
        cout << "2. Simulate LRU\n";
        cout << "3. Simulate OPTIMAL\n";
        cout << "4. Run All Strategies\n";
        cout << "5. Exit Program\n";
        cout << "--------------------------------------------------\n";
        cout << "Enter your choice: ";
        cin >> choice;

        switch (choice) {
            case 1: simulateFIFO(refs, frames); break;
            case 2: simulateLRU(refs, frames); break;
            case 3: simulateOptimal(refs, frames); break;
            case 4:
                simulateFIFO(refs, frames);
                simulateLRU(refs, frames);
                simulateOptimal(refs, frames);
                break;
            case 5:
                cout << "Program terminated.\n";
                break;
            default:
                cout << "Invalid choice. Please try again.\n";
        }

    } while (choice != 5);

    return 0;
}

```
