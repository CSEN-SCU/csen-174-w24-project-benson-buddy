Project by Varun Mangla, Jackson Lagerway, Abdullah Naveed, Wesley Tu, and Andrew Katchour.

A Chatbot Assistant geared around assisting users with navigating the Benson menu at SCU and determining the best meals for their needs.

The voiceflow-discord folder is comprised of code that we borrowed from online and used for testing purposes; we made some changes to try to get it to work with threads and our original replit, but, as it is right now, our discord implementation is not complete.

Additionally, voiceflow + replit + assistant API integration utilizes following template to facilitate the connections: https://replit.com/@bogdansaranchuk/custom-gpts-to-website-template?v=1#main.py . (Past versions of our code implemented more validating mechanisms and error checks but this resulted in a lot of issues running our code so we decided to simplify it as it worked best). Majority of the code behind our bot is seen in the natural language used to provide instructions for our bot due to the nature of the Assistant API tool we used.

Constructing the database in the form of a csv file was also a task that required trial and error and research in order to find the most effective file type to pass information to the bot. We utilized the code interpreter tool on the assistant API to analyze the csv file more deeply, compared to the regular retrieval method which looked through our original .docx file.

