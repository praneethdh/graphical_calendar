# 📅 Graphic Calendar

A C++ based **graphical calendar application** developed using the **BGI graphics library**.  
The project displays a monthly calendar with interactive mouse controls and also demonstrates the use of **Dynamic Programming** through a Travelling Salesman Problem (TSP) implementation.

---

## ✨ Features

- Graphical monthly calendar view
- Mouse-based navigation
  - Previous / Next Month
  - Previous / Next Year
- Accurate date calculation with leap year support
- Uses Zeller’s Congruence to determine starting weekday
- Travelling Salesman Problem (TSP) solved using Dynamic Programming
- Distance calculation using the Haversine formula
- Exit the application using the `ESC` key

---

## 🛠️ Technologies Used

- C++
- BGI Graphics Library (`graphics.h`)
- Mouse and keyboard event handling
- Dynamic Programming
- Computational geometry concepts

---

## ⚙️ How the Project Works

- The calendar is rendered using the BGI graphics library
- Navigation buttons are drawn on the screen and controlled using mouse clicks
- The starting day of each month is calculated mathematically
- A predefined set of geographic locations is used to compute the shortest travel path using TSP
- Distance between locations is calculated using latitude and longitude values

---

## ▶️ How to Run

### Requirements
- Turbo C++ / Dev-C++ / WinBGIm
- Properly configured `graphics.h`
