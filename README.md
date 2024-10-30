# Catlog-Coding
Problem Statement
In this assignment, you'll work on a simplified version of Shamir's Secret Sharing algorithm.

Consider an unknown polynomial of degree m. You would require m+1 roots of the polynomial to solve for the coefficients, represented as k = m + 1.
An unknown polynomial of degree m can be represented as:
f(x) = a_m x^m + a_{m-1} x^{m-1} + ... + a_1 x + c
Where:
f(x) is the polynomial function
m is the degree of the polynomial
a_m, a_{m-1}, ..., a_1, c are coefficients (real numbers)
a_m ≠ 0 (since it's the highest degree term, ensuring the polynomial is of degree m)
This representation shows that a polynomial of degree m is a sum of terms, where each term is a coefficient multiplied by a power of x. The highest power of x is m, and the powers decrease by 1 for each subsequent term until we reach the constant term c, which has no x.
The task is to find the constant term i.e, ‘c’ of the polynomial with the given roots. However, the points are not provided directly but in a specific format.
You need to read the input from the test cases provided in JSON format.
Sample Test Case:
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
    "6": {
        "base": "4",
        "value": "213"
    }
}
​
n: The number of roots provided in the given JSON
k: The minimum number of roots required to solve for the coefficients of the polynomial
k = m + 1, where m is the degree of the polynomial
Root Format Example:
"2": {
    "base": "2",
    "value": "111"
}
​
Consider the above root as (x, y):
x is the key of the object (in this case, x = 2)
y value is encoded with a given base
Decode y value: 111 in base 2 is 7
Therefore, x = 2 and y = 7
You can use any known method to find the coefficients of the polynomial, such as:
Lagrange interpolation
Matrix method
Gauss elimination
Solve for the constant term of the polynomial, typically represented as c.
Assignment Checkpoints:
1. Read the Test Case (Input) from a  separate JSON file
Parse and read the input provided in JSON format from a separate file, which contains a series of polynomial roots
2. Decode the Y Values
Correctly decode the Y values that are encoded using different bases
3. Find the Secret (C)
Calculate the secret c using the decoded Y values and any known method
Constraints:
All the coefficients a_m, a_{m-1}, ..., a_1, c are positive integers.
The coefficients are within the range of a 256-bit number.
The minimum number of roots provided (n) will always be greater than or equal to k (the minimum number of roots required).
The degree of the polynomial (m) is determined as m = k−1, where k is provided in the input.
  
Output: Print secret for both the testcases simultaneously.
Hint: Although you can't test your code against the test case in a testing environment, you can double-check it manually by solving the polynomial on paper and comparing the outputs.
Good luck!

Find the second testcase here.
{
"keys": {
    "n": 10,
    "k": 7
  },
  "1": {
    "base": "6",
    "value": "13444211440455345511"
  },
  "2": {
    "base": "15",
    "value": "aed7015a346d63"
  },
  "3": {
    "base": "15",
    "value": "6aeeb69631c227c"
  },
  "4": {
    "base": "16",
    "value": "e1b5e05623d881f"
  },
  "5": {
    "base": "8",
    "value": "316034514573652620673"
  },
  "6": {
    "base": "3",
    "value": "2122212201122002221120200210011020220200"
  },
  "7": {
    "base": "3",
    "value": "20120221122211000100210021102001201112121"
  },
  "8": {
    "base": "6",
    "value": "20220554335330240002224253"
  },
  "9": {
    "base": "12",
    "value": "45153788322a1255483"
  },
  "10": {
    "base": "7",
    "value": "1101613130313526312514143"
  }
}
Solution:I had used C programing for above problem statement
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>
#include <json-c/json.h>

// Function to convert a string in a given base to an integer
unsigned long long convert_base_to_decimal(const char* value, int base) {
    unsigned long long result = 0;
    unsigned long long base_power = 1;
    size_t len = strlen(value);
    
    for (int i = len - 1; i >= 0; i--) {
        char digit = value[i];
        if (digit >= '0' && digit <= '9') {
            result += (digit - '0') * base_power;
        } else if (digit >= 'a' && digit <= 'f') {
            result += (digit - 'a' + 10) * base_power;
        } else if (digit >= 'A' && digit <= 'F') {
            result += (digit - 'A' + 10) * base_power;
        }
        base_power *= base;
    }
    
    return result;
}

// Function to calculate the secret constant term c
unsigned long long calculate_secret(int n, int k, unsigned long long* x_values, unsigned long long* y_values) {
    unsigned long long c = 0;

    // Using Lagrange interpolation to find c
    for (int i = 0; i < k; i++) {
        unsigned long long term = y_values[i];
        for (int j = 0; j < k; j++) {
            if (i != j) {
                term *= x_values[j] / (x_values[j] - x_values[i]);
            }
        }
        c += term;
    }

    return c;
}

int main() {
    FILE *file;
    struct json_object *parsed_json;
    struct json_object *keys, *key_value, *base_value, *value_value;

    file = fopen("test_case.json", "r");
    if (!file) {
        fprintf(stderr, "Could not open file\n");
        return EXIT_FAILURE;
    }

    // Read the entire file into a string
    fseek(file, 0, SEEK_END);
    long length = ftell(file);
    fseek(file, 0, SEEK_SET);
    
    char *data = malloc(length);
    fread(data, 1, length, file);
    fclose(file);

    // Parse JSON data
    parsed_json = json_tokener_parse(data);
    free(data);

    json_object_object_get_ex(parsed_json, "keys", &keys);
    
    int n = json_object_get_int(json_object_object_get(keys, "n"));
    int k = json_object_get_int(json_object_object_get(keys, "k"));

    unsigned long long x_values[k];
    unsigned long long y_values[k];

    // Extract x and y values from JSON
    for (int i = 1; i <= n; i++) {
        char key[10];
        snprintf(key, sizeof(key), "%d", i);
        
        key_value = json_object_object_get(parsed_json, key);
        
        base_value = json_object_object_get(key_value, "base");
        value_value = json_object_object_get(key_value, "value");

        int base = json_object_get_int(base_value);
        const char* value_str = json_object_get_string(value_value);

        x_values[i - 1] = i; // x values are simply the keys
        y_values[i - 1] = convert_base_to_decimal(value_str, base);
    }

    // Calculate secret constant term c
    unsigned long long secret_c = calculate_secret(n, k, x_values, y_values);

    printf("Secret constant term c: %llu\n", secret_c);

    json_object_put(parsed_json); // Free memory allocated by JSON-C
    return EXIT_SUCCESS;
}
Use:gcc shamir.c -o shamir for compilation of the above code save file name as shamir.c
To Execute use ./shamir
Test Case 1:
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
  "6": {
      "base": "4",
      "value": "213"
  }
}
output:Calculated value
Test Case 2:
{
  "keys": {
      "n": 10,
      "k": 7
  },
  ...
}
Output:Calculated_value
