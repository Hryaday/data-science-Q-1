import requests
import json

# Configuration
GENERATE_WEBHOOK_URL = "https://bfhldevapigw.healthrx.co.in/hiring/generateWebhook/PYTHON"
SUBMIT_WEBHOOK_URL = "https://bfhldevapigw.healthrx.co.in/hiring/testWebhook/PYTHON"
USER_DATA = {
    "name": "John Doe",
    "regNo": "REG12347",
    "email": "john@example.com"
}

def generate_webhook():
    """Send POST request to generate webhook and return webhook URL and access token."""
    headers = {"Content-Type": "application/json"}
    response = requests.post(GENERATE_WEBHOOK_URL, json=USER_DATA, headers=headers)
    data = response.json()
    return data["webhook"], data["accessToken"]

def determine_sql_question(reg_no):
    """Determine which SQL question to solve based on the last digit of regNo."""
    last_digit = int(reg_no[-1])
    if last_digit % 2 == 0:
        return "Even question (Placeholder: SELECT * FROM employees WHERE salary > 50000)"
    else:
        return """SELECT 
    p.AMOUNT AS SALARY,
    CONCAT(e.FIRST_NAME, ' ', e.LAST_NAME) AS NAME,
    FLOOR(DATEDIFF('2025-05-11', e.DOB) / 365.25) AS AGE,
    d.DEPARTMENT_NAME
FROM 
    PAYMENTS p
    JOIN EMPLOYEE e ON p.EMP_ID = e.EMP_ID
    JOIN DEPARTMENT d ON e.DEPARTMENT = d.DEPARTMENT_ID
WHERE 
    DAY(p.PAYMENT_TIME) != 1
ORDER BY 
    p.AMOUNT DESC
LIMIT 1;"""

def submit_solution(webhook_url, access_token, sql_query):
    """Submit the SQL solution to the webhook URL with the access token."""
    headers = {
        "Authorization": access_token,
        "Content-Type": "application/json"
    }
    payload = {"finalQuery": sql_query}
    requests.post(webhook_url, json=payload, headers=headers)
    print("Solution submitted successfully!")

def main():
    # Step 1: Generate webhook
    webhook_url, access_token = generate_webhook()

    # Step 2: Determine and solve SQL question based on regNo
    sql_query = determine_sql_question(USER_DATA["regNo"])
    print(f"Generated SQL Query: {sql_query}")

    # Step 3: Submit the solution
    submit_solution(webhook_url, access_token, sql_query)

if __name__ == "__main__":
    main()
