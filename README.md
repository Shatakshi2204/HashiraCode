#include <iostream>
#include <string>
#include <vector>
#include <nlohmann/json.hpp>

using json = nlohmann::json;

// Decode function to convert base-n string to decimal number
unsigned long long decode_value(const std::string &value, int base) {
    unsigned long long result = 0;
    for (char c : value) {
        int digit = 0;
        if (c >= '0' && c <= '9')
            digit = c - '0';
        else if (c >= 'a' && c <= 'z')
            digit = c - 'a' + 10;
        else if (c >= 'A' && c <= 'Z')
            digit = c - 'A' + 10;
        result = result * base + digit;
    }
    return result;
}

// Lagrange interpolation function to compute polynomial value at x=0
double lagrange_interpolate(const std::vector<int> &x_points, const std::vector<unsigned long long> &y_points, int x) {
    int n = x_points.size();
    double result = 0.0;
    for (int i = 0; i < n; i++) {
        double term = y_points[i];
        for (int j = 0; j < n; j++) {
            if (j != i) {
                term *= double(x - x_points[j]) / (x_points[i] - x_points[j]);
            }
        }
        result += term;
    }
    return result;
}

int main() {
    // JSON data as raw string literal
    std::string json_text = R"(
{
  "keys": {
    "n": 4,
    "k": 3
  },
  "1": {
    "base": "10",
    "value": "4"
  },
  "2": {
    "base": "2",
    "value": "111"
  },
  "3": {
    "base": "10",
    "value": "12"
  },
  "4": {
    "base": "4",
    "value": "213"
  }
}
)";

    json inputData = json::parse(json_text);

    int n = inputData["keys"]["n"];
    int k = inputData["keys"]["k"];

    std::vector<int> x_points;
    std::vector<unsigned long long> y_points;

    for (int i = 1; i <= n; ++i) {
        std::string key = std::to_string(i);
        if (inputData.contains(key)) {
            int x = i;
            int base = std::stoi(inputData[key]["base"].get<std::string>());
            std::string val = inputData[key]["value"];
            unsigned long long y = decode_value(val, base);
            x_points.push_back(x);
            y_points.push_back(y);
            std::cout << "Root " << x << ": y (decoded) = " << y << std::endl;
        }
    }

    int points_to_use = std::min(k, static_cast<int>(x_points.size()));
    double secret_c = lagrange_interpolate(
        std::vector<int>(x_points.begin(), x_points.begin() + points_to_use),
        std::vector<unsigned long long>(y_points.begin(), y_points.begin() + points_to_use),
        0);

    std::cout << "Secret constant c (polynomial at x=0): " << secret_c << std::endl;

    return 0;
}
