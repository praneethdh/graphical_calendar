# graphical_calendar
#include <graphics.h>
#include <conio.h>
#include <stdio.h>
#include <vector>
#include <cmath>
#include <limits>
#include <algorithm>

// Define the number of locations (events) to visit
#define NUM_LOCATIONS 5

// Structure to hold latitude and longitude
struct Location {
    double lat;
    double lon;
};

// Function declarations
void drawCalendar(int month, int year);
int getDaysInMonth(int month, int year);
int getStartingDay(int month, int year);
void displayMonthName(int month, int x, int y);
void drawButton(int x, int y, int width, int height, const char* label);
void handleMouseClick(int& month, int& year);
double calculateDistance(const Location& loc1, const Location& loc2);
double tsp(int current, int visited, const std::vector<std::vector<double>>& dist, std::vector<std::vector<double>>& memo);
double tspDP(int start, const std::vector<Location>& locations);

int main() {
    int gd = DETECT, gm;
    initgraph(&gd, &gm, (char*)"");  // Use (char*)"" to suppress warnings

    int year = 2024;
    int month = 1;

    // Define locations with latitude and longitude (example locations)
    std::vector<Location> locations = {
        {37.7749, -122.4194}, // San Francisco, CA
        {34.0522, -118.2437}, // Los Angeles, CA
        {36.1699, -115.1398}, // Las Vegas, NV
        {40.7128, -74.0060},  // New York, NY
        {41.8781, -87.6298}   // Chicago, IL
    };

    while (1) {
        cleardevice();
        drawCalendar(month, year);

        // Draw navigation buttons
        drawButton(50, 400, 100, 30, "Prev Month");
        drawButton(200, 400, 100, 30, "Next Month");
        drawButton(350, 400, 100, 30, "Prev Year");
        drawButton(500, 400, 100, 30, "Next Year");

        // Handle mouse click
        handleMouseClick(month, year);

        // Compute TSP solution
        double result = tspDP(0, locations);
        printf("Optimal path cost: %.2f\n", result);

        // Exit the program on pressing ESC key
        if (kbhit()) {
            char ch = getch();
            if (ch == 27) break;  // ESC key
        }

        delay(100); // Add a small delay to prevent rapid input
    }

    closegraph();
    return 0;
}

void drawCalendar(int month, int year) {
    int daysInMonth = getDaysInMonth(month, year);
    int startDay = getStartingDay(month, year);

    int maxx = getmaxx();
    int maxy = getmaxy();

    // Draw the month name and year
    char title[20];
    sprintf(title, "%d", year);
    displayMonthName(month, maxx / 2 - 50, 20);
    outtextxy(maxx / 2 + 30, 20, title);

    // Draw the days of the week
    const char* daysOfWeek[7] = {"Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"};
    for (int i = 0; i < 7; i++) {
        outtextxy(i * (maxx / 7) + 10, 50, (char*)daysOfWeek[i]);
    }

    // Draw the dates
    int x = startDay * (maxx / 7);
    int y = 80;
    char date[3];

    for (int day = 1; day <= daysInMonth; day++) {
        sprintf(date, "%d", day);
        outtextxy(x + 10, y, date);
        x += (maxx / 7);
        if (x >= maxx) {
            x = 0;
            y += 40;
        }
    }
}

int getDaysInMonth(int month, int year) {
    int daysInMonth[] = {31, (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0) ? 29 : 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
    return daysInMonth[month - 1];
}

int getStartingDay(int month, int year) {
    int day = 1;
    int m = (month < 3) ? month + 12 : month;
    int y = (month < 3) ? year - 1 : year;

    int k = y % 100;
    int j = y / 100;

    int h = (day + (13 * (m + 1)) / 5 + k + k / 4 + j / 4 + 5 * j) % 7;

    // Convert Zeller's outcome to match our calendar
    int startDay = (h + 5) % 7; // 0 = Sunday, 1 = Monday, ..., 6 = Saturday
    return startDay;
}

void displayMonthName(int month, int x, int y) {
    const char* monthNames[] = {"January", "February", "March", "April", "May", "June",
                          "July", "August", "September", "October", "November", "December"};
    outtextxy(x, y, (char*)monthNames[month - 1]);
}

void drawButton(int x, int y, int width, int height, const char* label) {
    rectangle(x, y, x + width, y + height);
    outtextxy(x + 10, y + 8, (char*)label);
}

void handleMouseClick(int& month, int& year) {
    int x, y;
    if (ismouseclick(WM_LBUTTONDOWN)) {
        getmouseclick(WM_LBUTTONDOWN, x, y);

        // Next Month
        if (x >= 200 && x <= 300 && y >= 400 && y <= 430) {
            month++;
            if (month > 12) {
                month = 1;
                year++;
            }
        }
        // Previous Month
        if (x >= 50 && x <= 150 && y >= 400 && y <= 430) {
            month--;
            if (month < 1) {
                month = 12;
                year--;
            }
        }
        // Next Year
        if (x >= 500 && x <= 600 && y >= 400 && y <= 430) {
            year++;
        }
        // Previous Year
        if (x >= 350 && x <= 450 && y >= 400 && y <= 430) {
            year--;
        }
    }
}

double calculateDistance(const Location& loc1, const Location& loc2) {
    const double R = 6371.0; // Radius of Earth in kilometers

    double lat1 = loc1.lat * M_PI / 180.0;
    double lon1 = loc1.lon * M_PI / 180.0;
    double lat2 = loc2.lat * M_PI / 180.0;
    double lon2 = loc2.lon * M_PI / 180.0;

    double dlat = lat2 - lat1;
    double dlon = lon2 - lon1;

    double a = sin(dlat / 2) * sin(dlat / 2) +
               cos(lat1) * cos(lat2) *
               sin(dlon / 2) * sin(dlon / 2);
    double c = 2 * atan2(sqrt(a), sqrt(1 - a));

    return R * c;
}

double tsp(int current, int visited, const std::vector<std::vector<double>>& dist, std::vector<std::vector<double>>& memo) {
    int n = dist.size();
    if (visited == (1 << n) - 1) {
        return dist[current][0]; // Return to start
    }

    if (memo[current][visited] != -1) {
        return memo[current][visited];
    }

    double answer = std::numeric_limits<double>::infinity();
    for (int i = 0; i < n; i++) {
        if ((visited & (1 << i)) == 0) {
            double temp = dist[current][i] + tsp(i, visited | (1 << i), dist, memo);
            answer = std::min(answer, temp);
        }
    }

    memo[current][visited] = answer;
    return answer;
}

double tspDP(int start, const std::vector<Location>& locations) {
    int n = locations.size();
    std::vector<std::vector<double>> dist(n, std::vector<double>(n));
    std::vector<std::vector<double>> memo(n, std::vector<double>(1 << n, -1));

    // Initialize distances
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (i != j) {
                dist[i][j] = calculateDistance(locations[i], locations[j]);
            } else {
                dist[i][j] = 0;
            }
        }
    }

    return tsp(start, 1 << start, dist, memo);
}
