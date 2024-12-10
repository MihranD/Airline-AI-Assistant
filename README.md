# Multimodal AI assistant integrating Image and Sound generation
***

![Assitant](https://github.com/MihranD/Airline-AI-Assistant/blob/main/images/assistant.png)

## Project Organization
----------------------------------------------------------------------------------------------
    ├── .gitignore          <- Includes files and folders that we don't want to control
    |
    ├── images              <- Images for use in this project
    │   ├── assistant.png       <- Assistant image
    │   └── result.png          <- Result image
    |
    ├── main.ipynb          <- Main jupyter file that needs to be run
    |
    ├── requirements.txt    <- The required libraries to deploy this project. 
    |                       Generated with `pip freeze > requirements.txt`
    └── README.md           <- The top-level README for developers using this project.

## Project Introduction

`Tools` are an incredibly powerful feature provided by the frontier LLMs.
With tools, you can write a function, and have the LLM call that function as part of its response.
We're giving it the power to run code on our machine.

### Let's make a useful function

```
ticket_prices = {"london": "$799", "paris": "$899", "tokyo": "$1400", "berlin": "$499"}

def get_ticket_price(destination_city):
    print(f"Tool get_ticket_price called for {destination_city}")
    city = destination_city.lower()
    return ticket_prices.get(city, "Unknown")
```

There's a particular dictionary structure that's required to describe our function:

```
price_function = {
    "name": "get_ticket_price",
    "description": "Get the price of a return ticket to the destination city. Call this whenever you need to know the ticket price, for example when a customer asks 'How much is a ticket to this city'",
    "parameters": {
        "type": "object",
        "properties": {
            "destination_city": {
                "type": "string",
                "description": "The city that the customer wants to travel to",
            },
        },
        "required": ["destination_city"],
        "additionalProperties": False
    }
}
```

```
tools = [{"type": "function", "function": price_function}]
```

## Getting OpenAI to use our Tool

There's some fiddly stuff to allow OpenAI `to call our tool`.
What we actually do is give the LLM the opportunity to inform us that it wants us to run the tool.
Here's how the new chat function looks:

```
def chat(history):
    messages = [{"role": "system", "content": system_message}] + history
    response = openai.chat.completions.create(model=MODEL, messages=messages, tools=tools)
    image = None
    
    if response.choices[0].finish_reason=="tool_calls":
        message = response.choices[0].message
        response, city = handle_tool_call(message)
        messages.append(message)
        messages.append(response)
        image = artist(city)
        response = openai.chat.completions.create(model=MODEL, messages=messages)
        
    reply = response.choices[0].message.content
    history += [{"role":"assistant", "content":reply}]

    # Comment out or delete the next line if you'd rather skip Audio for now..
    talker(reply)
    
    return history, image
```

We have to write that function handle_tool_call:

```
def handle_tool_call(message):
    tool_call = message.tool_calls[0]
    arguments = json.loads(tool_call.function.arguments)
    city = arguments.get('destination_city')
    price = get_ticket_price(city)
    response = {
        "role": "tool",
        "content": json.dumps({"destination_city": city,"price": price}),
        "tool_call_id": message.tool_calls[0].id
    }
    return response, city
```

```
# More involved Gradio code as we're not using the preset Chat interface!
# Passing in inbrowser=True in the last line will cause a Gradio window to pop up immediately.

with gr.Blocks(js=force_dark_mode) as ui:
    with gr.Row():
        chatbot = gr.Chatbot(height=500, type="messages")
        image_output = gr.Image(height=500)
    with gr.Row():
        entry = gr.Textbox(label="Chat with our AI Assistant:")
    with gr.Row():
        clear = gr.Button("Clear")

    def do_entry(message, history):
        history += [{"role":"user", "content":message}]
        return "", history

    entry.submit(do_entry, inputs=[entry, chatbot], outputs=[entry, chatbot]).then(
        chat, inputs=chatbot, outputs=[chatbot, image_output]
    )
    clear.click(lambda: None, inputs=None, outputs=chatbot, queue=False)

ui.launch(inbrowser=True)
```

# Our Agent Framework

The term **Agentic AI** and Agentization is an umbrella term that refers to a number of techniques, such as:

1. Breaking a complex problem into smaller steps, with multiple LLMs carrying out specialized tasks
2. The ability for LLMs to use Tools to give them additional capabilities
3. The `Agent Environment` which allows Agents to collaborate
4. An LLM can act as the Planner, dividing bigger tasks into smaller ones for the specialists
5. The concept of an Agent having autonomy / agency, beyond just responding to a prompt - such as Memory
    
![Result](https://github.com/MihranD/Airline-AI-Assistant/blob/main/images/result.png)

## Conclusion

Tools in OpenAI's language models allow the LLM to run functions and execute code, enhancing its ability to perform dynamic tasks and provide more personalized, real-time responses. This makes the model more interactive and powerful.

## How to run the app

Follow these steps to set up and run the application:

1. **Create a `.env` file**  
   Add your OpenAI API key to the `.env` file in the following format:  
   ```plaintext
   OPENAI_API_KEY=sk-proj-blabla
   ```
   
2. **Setup virtual envirenment**  
   Run the following command to setup virtual envirenment:  
   ```bash
   python3 -m venv venv  # Create a virtual environment named 'venv'
   source venv/bin/activate  # Activate the virtual environment (Linux/Mac)'
   .\venv\Scripts\activate   # Activate the virtual environment (Windows)'
   ```

3. **Install Dependencies**  
   Run the following command to install all required dependencies:  
   ```bash
   pip install -r requirements.txt
   ```

4. **Open and Run the Notebook**  
   Open the `main.ipynb` file in Jupyter Notebook and execute the cells to run the application.

