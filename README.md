# RAG Chatbot LLM Project Guide Notes

## Preface: Context to this Document

-  I had something happen to my github repo while going through this tutorial, and I wasn't able to recover it. I've provided my code and the files I used in this github repo, but items like the commit history and other comments were lost.

-  Thankfully, I wrote out notes on a google doc when I was going through this tutorial. It highlights some of the struggles, questions, thoughts, and solutions I went through when completing this project. While this doesn't make up for the lost commit history or comments, I hope this might be of use. I apologize for the lack of professionalism.

## Part 1: Initial Setup and MongoDB Configuration

1. Created a new python virtual environment using “python3 -m venv jaclang”, creating a virtual environment where I can install pip packages that aren’t debian backed.
    - To get into it, do “source jaclang/bin/activate”, to leave, do “deactivate”

2. Used pip install to install jaclang and jacloud.

3. Installing MongoDB Community edition for Debian (I’m using WSL2). (https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-debian/)
    - When you run with the set datapath (mine is /home/michkang/res/mongolog), only then you can only do mongosh.. (You had to have been running both at the same time) Took me a little while to figure out.
    - Did rs.initiate() to start the process.
        - I guess, I need to set the datapath and run, then open another window to actually use mongosh. This datapath setup thing needs to be running in another window for us to run mongosh.

4. Downloading Docker from “https://www.docker.com/products/docker-desktop/” .
    - Downloading the Docker implementation for windows, then activating it for WSL. It looks like I need to do this rather than the actual linux implementation due to the fact that I can’t run a GNOME desktop environment in my wsl tab.
    - By default, Docker Desktop stores the data for the WSL 2 engine at C:\Users\[USERNAME]\AppData\Local\Docker\wsl. If you want to change the location, for example, to another drive you can do so via the Settings -> Resources -> Advanced page from the Docker Dashboard.

5. I’ve set up docker, now I’m pulling the mongoDB image from dockerhub. 
    - In between this, I’ve logged into docker through the terminal.

6. I’ve run the image as a container. This will start a mongoDB container with the name mongodb and expose port 27017 to the host machine.

7. Enter docker ps to get the lists of running containers.

8. I already have mongosh installed, so I have connected the port to the container with “mongosh –port 27017”
    - I’m running the mongo shell to initiate the replica set. Using rs initiate with the default configuration.
    - I got a little different output than what is called for, but I think it’s ok. I got the Ok:1, so I think it’s alright. I got a code about the cluster timestamp, but I think it’s just additional information.

9. Installed the jac extension on vscode.

10. Making the file named server.jac, and defining the code.

11. Attempting to serve the code to Jac Cloud by running a certain command. However, my localhost doesn’t look like the defined one. It just says “Not Found”.
    - Found the error. When you’re in the localhost, just put “/docs” after the “localhost:8000” to find the page referenced in the instructions.
    - DATABASE_HOST=mongodb://localhost:27017/?replicaSet=my-rs jac serve server.jac
    
12. Authenticating the API in Jac Cloud by registering a user.
    - Once you register, we need to get the token from logging in. Wwe then use the interactive api we created to authenticate and use our token for requests. Once we do this, we should get the message that we made in server.jac, which is hello world.
        - I was having a really hard time because my token wouldn’t authenticate. I thought it was a copying and pasting error like ASCII or something, but it turns out when it says <TOKEN> in the project spec, you’re supposed not include the angle brackets when pasting in the token. Yeah this is my fault.
    - Went to the docs and used the authorization header to register the personnel using the bearer access token.

## Part 2: Dependencies and Environment Setup

1. Installing new dependencies using pip.
    - Okay, I guess that my python is 3.11.2, so I can’t install these things. However, I can’t seem to find a clean way to upgrade my python to 3.12.
        - I accidentally made my venv in python 3.11.2, so I'll have to delete the venv and restart. This is a little disappointing.
    - I have installed python 3.13.0 . 
    - After recreating, it wasn't that bad. Everything stayed for the most part and I just had to run some updates to get back up to speed.
    - It still doesn’t work. The output says I'm missing items for "onxxruntime" . I think I'll have to downgrade to python 3.12.

2. Creating the client.jac file
    - After switching to python 3.12 and making a new venv again, it looks like the dependencies work.

3. It works! Localhost 8501 opens a port that has a chatbot interface that only says hello world. However, it works! I’m really glad.

4. So, we now know what a RAG is, and our simple steps to configure one. I’ve created the rag.jac file with our current engine.
    - Basically, we load documents from the specified directory, by using a class named PyPDFDirectoryLoader. We then split the document using a method in the class RecusirveCharacterTestSplitter from the langchain_text_splitters module. We split them into chunks. We use a function called get_embedding_function to initialize the model we got from Ollama. We use the add_chunk_id method to generate unique ID’s for the chunks based on source and the page number. We now use add_to_chrom to add the chunks to the chroma vector storage. Finally, we use the get_from_chroma method to retrieve candidate responses based on the query from the chroma vector store.

5. Now, we’re downloading a suitable model from Ollama. I’m using the linux installation method. It seems that we just say “ollama" in the cli to run the api. 

6. Let's Set up the ollama embeddings! 
    - Okay, running into errors. We want to pull the nonic-embed-text model from ollama, but it says we can’t. "Error: could not connect to the ollama app, is it running?"
    - In the help option, it has a option called “ollama serve”, which just says start ollama. I did it, and it’s writing to the terminal. I should save this also: 
    Couldn't find '/home/michkang/.ollama/id_ed25519'. Generating new private key. Your new public key is: 

    ssh-ed25519 ----------------------------------------------------------------------------

    - It seems that I have to run "ollama serve" before using any of the other possible arguments. Seeing how things are ran from part 1, this makes sense.

7. We’re in, we’ve downloaded the model, and we’ve let it serve. Now, the tutorial references a documents directory that we're supposed to add to. I hadn't seen this before, so I just decided to create it.
    - Made a directory called docs in the root of my project and then added the provided pdf.

8. Looks like I need an API key from OpenAI to continue. We’re adjusting the server.jac file to actually incorporate a real model. We’re going to use OPenAI model gpt-4o as our model here with the api key.
    - Here is my key. I named it jaclang LLM:
    sk-B-------------------------------------------------------------------------------------------

9. We’re now setting up the LLM. Using the api key I got from openAI, we’re setting up the server.jac file to work with the openAI key, and the hold the session data.
    - We made a node that holds a unique session identifier, holds the chat history, the state of the session, and has the ability to take the current message, chat history, agent role, and the retrieved document context as inputs. Then, uses the LLM to generate a response base on these inputs. This is pretty fucking cool bro.
    - We now set up an interaction called walker that initializes a session and generates the responses to the user queries.	
        - A walker is a mechanism that traverses the graph. We move from node to node, executing abilities and interacting with the data stored in the nodes. For this specific one, the interact walker, we handle user interactions and generate responses.
        - It holds the users message, the unique session identifier, and then makes a session based on the session ID. if it doesn’t exist already, it creates a new session node. It’s triggered on root entry. In every graph, there is a root node that serves as the start. A walker can be spawned on and traverse to any node in the graph. It doesn’t have to be the root node, but it can be spawned on the root node.

10. The logic flow: we add the users message to the chat history, we use the RAG engine to retrieve responses based on the users message, the LLM model generates a response based on the user message, history of the chat, agent role, and retrieved context. The assistance is added to the chat history. This response is reported back to the frontend. It uses a special keyword called report. Kinda like a return statement, but doesn’t stop the execution of the walker. It’s just going to add whatever is reported to the response object back to the frontend.

11. To summarize, we define a session node to store chat history and status of session. The session node has an ability called LLM_chat that uses the LLM model to generate responses based on the chat history, role of agent and context. We then define an interactive walker that initializes a session and genreates responses to user queries. The walker uses the rag_engine object to retrieve candidate responses and the llm_chat ability to generate the final response.
    - We can now run it using this! DATABASE_HOST=mongodb://localhost:27017/?replicaSet=my-rs jac serve server.jac

12. I think that I’m supposed to pip install openAI, which is giving my laptop a really hard time when downloading. I’m not gonna use OpenAI, but I’m just going to get a model from ollama called llama3.1 instead. I'm more interested in running a local model and using ollama, instead of using just an API Key.
    - It doesn’t work. I’m not sure what's happening but I’m missing a module.

13. I’ve successfully download llama3.1 this time, as apparently I didn’t fully download it and I got an error instead. It turns out that when you pull a model from ollama, you’re just supposed to spam the pull requests until it works.
    - Although, I still have the same issue. I’m not quite sure what's going on, but it seems like that module can’t connect to the model for some reason.
    - I guess since I didn’t correctly install ollama using pip, it just didn’t work. I just had to run "pip install ollama" again to like maybe verify the installation? Anyhow, it fixed the error.

14. It works! Lets move onto part 3.

## Part 3: Dialogue Capabilities

1. Onto part 3! We're now integrating dialogue capabilities. We’re adding an enum to differentiate states. The states are RAG, which is used to get retrievable information in the documents to respond to user queries. The QA state is used when the context is enough for an answer.

2. Then, we implement a router node for determining RAG or QA to handle a query.

3. Since the router node has an ability called classify to the user query, and then puts it into a chat type by using the “by llm” feature from MTLLM.
    - We classify as Ragchat and QA chat, which is an extension of the chatnode. Ragnode is used for the RAG engine, and QAnode is used for a simple question answer model. THey can both respond to the user query using the respective model.

4. Now, we’ll add a new walker called infer. It holds the logic for routing the user query to the appropriate dialogue model based on the classification from the router node.
    - Now we have an ability called init_router, that initializes the router node and creates and bridges nodes RagChat and QAchat. These nodes can handle both models. 
    - We add an ability called route that classifies the user query using the route node, and then routes it to the appropriate node on the classification.
5. We update node sessions to maintain a record of previous conversation history so the chatbot can have a continuous back and forth conversion with previous memory of interactions.

6. Core Changes:
    - New ChatType enum with two types: RAG and QA
    - New chat ability in Session node (replacing the old interact walker entry)
    - Interaction flow: interact walker → Session node → infer walker → appropriate model
- New Components:
    - Router node: Classifies user queries
    - infer walker: Routes queries based on classification
- Two chat model nodes:
    - RagChat: Handles RAG-based responses
    - QAChat: Handles simple Q&A responses
- Flow:
    - User query → Session node → Router classification → Appropriate model (RAG or QA) → Response back to frontend

7. After I pasted the implementations in, I’m getting a compilation error with “name “infer” is not defined”.   
    - I forgot that you need to have function declaration because this language is “top down”, IE, if you don’t have it declared before its called, it doesn’t know it exists. My error is just moving things around until they’re declared fully.
    - And, I accidentally changed the “Session” node incorrectly, this is my bad.
8. We’re done!


## Conclusions:

- Honestly, this was a really fun tutorial. I know I didn't do much coding, but seeing how these different softwares worked together that I was mildy unfamiliar with was pretty cool. It was really interesting to see how the jaclang worked in usison to create the RAG engine, and the context storing for the LLM. The syntax and other portions were a bit confusing for me to understand at first, but it actually was super convenient at the end. Pretty amazing stuff that we can just let walkers do their own thing and route nodes for us. Getting some of the software and compatability to work was somewhat of a pain, but it's to be expected when working with new ideas.