import os
import json
import openai
from time import sleep
from argparse import ArgumentParser, Namespace

from selenium import webdriver
from selenium.common import exceptions
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.support.ui import WebDriverWait
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.support import expected_conditions as EC


class Bcolors:
    HEADER = '\033[95m'
    OKBLUE = '\033[94m'
    OKCYAN = '\033[96m'
    OKGREEN = '\033[92m'
    WARNING = '\033[93m'
    FAIL = '\033[91m'
    ENDC = '\033[0m'
    BOLD = '\033[1m'
    UNDERLINE = '\033[4m'


parser = ArgumentParser(description="WhatsApp AI Chatbot")

parser.usage = 'python main.py --name "Chat Name" --choice 1'

parser.add_argument('-n', '--name',
                    type=str,
                    dest='name',
                    required=True,
                    metavar='',
                    help='Name of the person you wish to send a message to'
)
parser.add_argument('-c', '--choice', type=int,
                    dest='user_choice',
                    metavar='',
                    choices=[1, 2],
                    default=2,
                    help='Messaging option:\n'
                         '1. Reply manually\n'
                         '2. Generate AI response',
)

args: Namespace = parser.parse_args()

# Current user's home directory
home_dir = os.path.expanduser("~")
user_data_dir = os.path.join(home_dir, "AppData", "Local", "Google", "Chrome", "User Data", "Default")

chrome_options = webdriver.ChromeOptions()
chrome_options.add_experimental_option("detach", True)
chrome_options.add_argument(f'--user-data-dir={user_data_dir}')
chrome_options.add_argument('--profile-directory=Default')

driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=chrome_options)

driver.get("https://web.whatsapp.com/")
driver.maximize_window()

wait = WebDriverWait(driver, 20)
try:
    wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, "div.k8VZe")))
except exceptions.TimeoutException:
    print(Bcolors.WARNING + "Login took longer than expected" + Bcolors.ENDC)


def scroll_to_bottom():
    last_height = driver.execute_script("return document.body.scrollHeight")
    while True:
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        sleep(1)
        new_height = driver.execute_script("return document.body.scrollHeight")
        if new_height == last_height:
            break
        last_height = new_height


# Call the scroll_to_bottom function to scroll to the bottom of the page
scroll_to_bottom()


def input_message():
    print(Bcolors.HEADER + "Please enter your message, and use the symbol '~' on a new line to indicate the end of the message.")
    print("For example: Hi, this is a test message\n~" + Bcolors.ENDC)
    print("Your message (press Enter to finish): ")

    message = []

    while True:
        temp = input()

        if temp.strip() == "~":
            break

        message.append(temp)

    return "\n".join(message)


def get_last_message(person_name):

    contact_name_selector = f'//span[@title="{person_name}"]'
    chat = driver.find_element(By.XPATH, contact_name_selector)
    chat.click()

    sleep(2)
    try:
        # Locate the message elements within the chat window
        message_elements = driver.find_elements(By.CSS_SELECTOR, '.message-in, .message-out')

        # Retrieve the last message element
        last_message_element = message_elements[-1]

        # Extract the text content of the last message, excluding the time stamp
        last_message = last_message_element.find_element(By.CSS_SELECTOR, '.selectable-text').text

        print(Bcolors.OKGREEN + "Last Message:", last_message + Bcolors.ENDC)

        # return the last message
        return last_message

    except IndexError as e:
        print(Bcolors.WARNING + "There are no recent messages." + Bcolors.ENDC)


def get_api_key():
    with open('api_key.json', 'r') as file:
        config = json.load(file)

    return config['api_key']


def generate_ai_response(args: str):

    try:
        openai.api_key = get_api_key()

        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[
                {
                    "role": "user",
                    "content": args
                }
            ]
        )

        return response.choices[0].message.content

    except openai.OpenAIError as e:
        # Handle OpenAI API errors
        print(Bcolors.FAIL + f"OpenAI API Error: {str(e)}" + Bcolors.ENDC)
        return None


def main(person_name, callback_func=None):
    # Select contact element based on person_name
    contact_name_selector = f'//span[@title="{person_name}"]'
    contact_elements = driver.find_elements(By.XPATH, contact_name_selector)

    for contact_element in contact_elements:
        if contact_element.get_attribute("title") == person_name:
            contact_element.click()

            if callback_func:
                message_input = input_message()
                message_input_xpath = driver.find_element(By.XPATH, '//footer//p')
                message_input_xpath.send_keys(message_input, Keys.ENTER)
                print(Bcolors.OKGREEN + "Message sent successfully.")

            if not callback_func:
                # Get the last message from the person
                last_message = get_last_message(person_name)

                # Generate AI response using chatgpt
                ai_response = generate_ai_response(last_message)

                message_input_xpath = driver.find_element(By.XPATH, '//footer//p')
                message_input_xpath.send_keys(ai_response, Keys.ENTER)
                print(Bcolors.OKCYAN + f"ChatGPT: {ai_response}")
                print(Bcolors.OKGREEN + "Message sent successfully.")
        break


contact_name = args.name

contact_found = False

while not contact_found:
    contact_name_selector = f'//span[@title="{contact_name}"]'
    contact_elements = driver.find_elements(By.XPATH, contact_name_selector)

    if len(contact_elements) > 0:
        contact_found = True
    else:
        print(Bcolors.FAIL + f"Contact with the name {Bcolors.BOLD}'{contact_name}'{Bcolors.ENDC} not found. Please try again." + Bcolors.ENDC)
        contact_name = input(Bcolors.OKBLUE + "Enter the name of the contact you want to message:\n" + Bcolors.ENDC)

print(Bcolors.OKGREEN + f"Contact with the name '{contact_name}' found!" + Bcolors.ENDC)


if args.user_choice:
    if args.user_choice == 1:
        main(contact_name, callback_func=input_message)
    if args.user_choice == 2:
        main(contact_name)
else:
    print(Bcolors.FAIL + "Invalid choice. Please enter a valid option." + Bcolors.ENDC)

print("Browser will be closed in 10 seconds.")
sleep(10)
driver.quit()
