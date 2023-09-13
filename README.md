# **WhatsApp Chatbot**

This script allows you to automate messaging on WhatsApp using Selenium and interact with OpenAI's GPT-3.5-turbo model to generate AI responses.

## **Prerequisites**

Before running the script, make sure you have the following dependencies installed:

- Python 3.x
- Linux : `pip3 install -r requirements.txt`
- Windows: `pip install -r requirements.txt`

**Note**: The `webdriver_manager` package simplifies the process of managing and downloading the appropriate WebDriver executable. When using the `webdriver.Chrome()` constructor, it will automatically download the appropriate ChromeDriver if it's not already loaded. If the driver is already present, it finds it in the cache and uses it directly.

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
```

This eliminates the need for manually setting the `executable_path` and managing the ChromeDriver version. The `webdriver_manager` package takes care of it for you, making the setup process easier and more convenient.

## **Usage**

1. Run the script by executing the following command:
      - `python main.py -n "chat name" -c [1, 2]`
      - `python main.py --name "chat name" --choice [1, 2]`     
3. Scan the QR code displayed in the console using your WhatsApp mobile app to log in.
4. Enter the name of the contact you want to message when prompted.
5. Choose an option:
   - Option 1: Reply yourself - You can manually type the message you want to send.
   - Option 2: Let us handle it for you - The script will retrieve the last message from the selected contact and generate an AI response using OpenAI's GPT-3.5-turbo model.
6. Follow the instructions on the console to enter your message or press Enter to finish.
7. The script will send the message or the AI-generated response to the selected contact.
8. The browser will be automatically closed after 10 seconds.

**Note: Make sure you have an active internet connection and the person you want to message is visible in your WhatsApp chat list.**

## **Important Note**

- Use this script responsibly and in accordance with WhatsApp's terms of service. Automated messaging may violate WhatsApp's policies, so it's recommended to use this script for personal and non-commercial purposes only.
- Remember to respect privacy and consent. Always obtain permission from the recipients before sending automated messages.
- Be cautious while interacting with OpenAI's GPT-3.5-turbo model and ensure that the generated responses align with ethical guidelines and standards.

## **Disclaimer**

This script is provided as-is without any warranty. Use it at your own risk. The script author and OpenAI shall not be held responsible for any misuse or damages caused by this script.
