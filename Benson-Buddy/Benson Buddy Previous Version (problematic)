import os
from time import sleep
from typing import Counter
from packaging import version
from flask import Flask, request, jsonify
import openai
from openai import OpenAI
import functions
import tenacity
import json
import signal
from array import array
import itertools


class TimeOutException(Exception):
  pass


def timeout_handler(signum, frame):
  raise TimeOutException


def check_OpenAI_Key():
  # Check OpenAI version is correct
  required_version = version.parse("1.1.1")
  current_version = version.parse(openai.__version__)
  global OPENAI_API_KEY  # We can make this local, but the program looks better if it's global. Maybe change...
  OPENAI_API_KEY = os.environ['OPENAI_API_KEY']
  if current_version < required_version:
    raise ValueError(f"Eglrror: OpenAI version {openai.__version__}"
                     " is less than the required version 1.1.1")
  else:
    print("OpenAI version is compatible.")
    init_Client(OPENAI_API_KEY)


def init_Client(OPENAI_API_KEY):
  # Start Flask app

  app = Flask(__name__)
  print("1")
  # Init client
  client = OpenAI(
      api_key=OPENAI_API_KEY
  )  # should use env variable OPENAI_API_KEY in secrets (bottom left corner)
  print("2")
  # Create new assistant or load existing
  assistant_id = functions.create_assistant(client)
  print("3")
  app.run(host='0.0.0.0', port=8080)
  conversation_Thread(client, assistant_id, app)


def print_Response(response):
  print(f"Assistant response: {response}")  # Debugging line
  return jsonify({"response": response})


def validate_Input(user_input, knowledge_dict):
  # sample code
  # So, what I want to do here is search for any keywords in the user input, try and find a match in the knowledge_dict (without case sensitivity), and then see if I can find a match for the parameters.
  # If I can't find a match, then I move on to asking the user whether they'd like "hallucinated" input
  # Based on the results, return true or false -> can use in the convo thread function for further work
  # Sample food items; Praline Latte, Chocolate Smoothie, Chocolate Cake, Coffee, Coffee with Milk, Coffee with Milk and Sugar
  # Implement Nutrition API if user asks for hallucinated output
  # I'll figure this out... Want to assign some sort of score saying how close user input is to the food item, and then return the best match.
  # Need to decide whether doing this by character or by word is better <- by character allows for typos, while by word reduces the risk of picking the wrong item.

  dict_set = set(knowledge_dict.keys())
  dict_list = list(knowledge_dict.keys())
  l_dict_list = [word.lower() for word in dict_list]
  lower_dict_list = [y for x in dict_set
                     for y in x.split(' ')]  # print for debugging purposes?
  lower_dict_list = [word.lower() for word in lower_dict_list]
  # print(lower_dict_list)
  user_words = user_input.split(" ")
  lower_user_list = [word.lower() for word in user_words]
  u_list = itertools.tee(lower_user_list)
  d_list = itertools.tee(lower_dict_list)
  # print(lower_user_words)
  Flag = 0
  word_store = "to be overwritten :("  # stores user food option
  for u_word, next_u_word, nn_u_word, nnn_u_word in u_list:
    for d_word, next_d_word, nn_d_word, nnn_d_word in d_list:
      word_store = u_word
      if u_word == d_word and next_u_word == next_d_word:  # better case (only doing 2 matches, can do more...) edit: baseline is two, adding more if we get them
        Flag = 1
        word_store += next_u_word
        if (nn_u_word == nn_d_word):
          word_store += nn_u_word
          if (nnn_u_word == nnn_d_word):
            word_store += nnn_u_word
        break
      elif (u_word == d_word):  # worse case
        Flag = .5
        break
    if (Flag > 0):
      break
  # Decision time...
  if (Flag == .5):
    Flag = 1 if word_store in l_dict_list else 0  # basically, OK if it's a solo item ("Latte") and bad if it's not
  if (Flag == 0):
    # User input could not be matched at all...
    word_store = "empty"
    return [False, word_store]  # no dice
  # User input could be matched at least somewhat...
  # Parameter matching portion
  list_index = l_dict_list.index(
      word_store)  # getting the index of the matched item
  word_store = dict_list[
      list_index]  # converting it to the proper formatting for dict indexing
  param_list = list(knowledge_dict[word_store].keys())
  l_param_list = [word.lower() for word in param_list]
  for word in lower_user_list:
    if word in l_param_list:
      return [True, word_store]  # True, food item, parameter
  return [False, word_store]  # False, food item


def conversation_Thread(client, assistant_id, app):
  print("4")
  # Start conversation thread
  knowledge_json = open('knowledge.json')  # Open the json file
  print("4")
  knowledge_dict = json.load(knowledge_json)  # Store entries in a dictionary
  print("4")
  # print knowledge_dict if we want to debug

  @app.route('/start', methods=['GET'])
  def start_conversation():
    print("Starting a new conversation...")  # Debugging line
    thread = client.beta.threads.create()
    print(f"New thread created with ID: {thread.id}")  # Debugging line
    return jsonify({"thread_id": thread.id})

  print("4")
  # Generate response
  @app.route('/chat', methods=['POST'])
  def chat():
    data = request.json
    thread_id = data.get('thread_id')
    user_input = data.get('message', '')
    if not thread_id:
      print("Error: Missing thread_id")  # Debugging line
      return jsonify({"error": "Missing thread_id"}), 400

    print(f"Received message: {user_input} for thread ID: {thread_id}"
          )  # Debugging line
    # VALIDATE USER INPUT START
    #returnlist = validate_Input(user_input, knowledge_dict)  # Parse user input
    #if returnlist[0] is False and returnlist[1] != "empty":  # We want to make sure that this
    # ONLY works on items that are in the db but don't have requisite params
    #print("Error: Invalid input")
    #response = "Was unable to fetch information for " + word_store + "."
    #response += " Hence, we can't guarantee the validity of the information provided."
    #print_Response(response)
    #VALIDATE USER INPUT END
    # Add the user's message to the thread
    print("1000")
    client.beta.threads.messages.create(thread_id=thread_id,
                                        role="user",
                                        content=user_input)

    # Run the Assistant
    print("10")
    run = client.beta.threads.runs.create(thread_id=thread_id,
                                          assistant_id=assistant_id)
    # @retry(wait=wait_exponential(multiplier=1, max=60), stop=stop_after_attempt(6))
    print("11")
    # Change the behavior of SIGALRM
    #signal.signal(signal.SIGALRM, timeout_handler)

    # Check if the Run requires action (function call)
    #while True:
    # Run timer check to make sure that the chatbot does not hang. If the chatbot does not respond within 30 seconds, it will be restarted.
    # signal.alarm(30)
    #try:
    # run_status = client.beta.threads.runs.retrieve(thread_id=thread_id,                                                run_id=run.id)
    #except TimeOutException:
    #response = "Sorry, I'm taking too long to respond. Please try again."
    #print_Response(response)
    #client.close()
    #init_Client(OPENAI_API_KEY)
    #exit(
    #)  # something tells me that this line might kill the program as a whole...
    #print(f"Run status: {run_status.status}")
    #if run_status.status == 'completed':
    #  break
    #sleep(1)  # Wait for a second before checking again

    # Retrieve and return the latest message from the assistant
    # print("11")
    messages = client.beta.threads.messages.list(thread_id=thread_id)
    response = messages.data[0].content[0].text.value
    print_Response(response)


check_OpenAI_Key()  # runs program
