## Enhanced Flask Server for Hydroponic System Management with LINE, GPT-4, and InfluxDB

This code creates a Flask web server that integrates LINE Bot, OpenAI's GPT-4, and InfluxDB for managing a hydroponic system. The server receives messages (text or images) from LINE users, processes them using GPT-4 (with contextual information like documents and sensor data), and sends back appropriate responses.

### Key Functionalities:

- **LINE Bot Integration:**
    - Receives text or image messages from users via the LINE webhook.
    - Sends responses using GPT-generated messages or image descriptions.
    - Utilizes the `line-bot-sdk` to handle message parsing and sending.
- **OpenAI GPT-4:**
    - Handles chat completions based on user input and chat history.
    - Chat history resets every 10 minutes to manage context length and prevent excessive token usage.
- **InfluxDB Integration:**
    - Fetches sensor data from InfluxDB for the hydroponic system.
    - Retrieves data for the last 10 days, formats it into a list, and stores it as a global variable.
- **Image Handling:**
    - Saves images received from LINE locally using temporary files.
    - Converts images to base64 and sends them to GPT-4 for image description.

### Example Enhancements:

- **Add More Document Information:**
    - The `docs` variable can hold more detailed farming-related information for richer GPT-4 responses.
- **Enhance Sensor Data Processing:**
    - Process or visualize the sensor data fetched from InfluxDB before presenting it to the user.
- **Optimize Image Handling:**
    - Handle different image types and formats more robustly.
- **Security:**
    - Store sensitive information like API tokens and keys using environment variables instead of hardcoding.

### Refactored Code Sections:

**1. Configuration:**

Load API tokens securely from environment variables:

```python
import os

lineaccesstoken = os.getenv("LINE_ACCESS_TOKEN")
openai_api_key = os.getenv("OPENAI_API_KEY")
influxdb_token = os.getenv("INFLUXDB_TOKEN")

# ... (rest of the initialization code for LINE Bot, OpenAI, InfluxDB)
```

**2. Fetching Sensor Data:**

Make bucket and organization configurable via environment variables:

```python
bucket = os.getenv("INFLUXDB_BUCKET", "Hydropolic-Farm")
org = os.getenv("INFLUXDB_ORG", "Ubru Farm")
query = f"""
    from(bucket: "{bucket}")
    |> range(start: {start_time_str}, stop: {end_time_str})
    |> filter(fn: (r) => r["_field"] == "Atmophere-Humidity" 
                      or r["_field"] == "Atmophere-Temperature" 
                      or r["_field"] == "Light" 
                      or r["_field"] == "Water-Temperature-Box-A" 
                      or r["_field"] == "Water-Temperature-Box-B")
    |> yield(name: "mean")
"""

# ... (rest of the InfluxDB data fetching code)
```

**3. Improved Error Handling:**

Implement robust error handling for LINE message processing:

```python
def event_handle(event):
    try:
        user_id = event["source"]["userId"]
        msg_id = event["message"]["id"]
        msg_type = event["message"]["type"]
    except KeyError:
        print("Error: Cannot get userId or message details")
        return

    if msg_type == "text":
        msg = event["message"]["text"]
        print(f"Received text message: {msg}")
        try:
            response = send_message(msg)
            messaging_api.push_message(
                PushMessageRequest(to=user_id, messages=[TextMessage(text=response)])
            )
        except Exception as e:
            print(f"Error sending message: {e}")

    elif msg_type == "image":
        try:
            image_path = save_image_locally(msg_id)
            if image_path:
                base64_image = encode_image(image_path)
                description = send_message(base64_image, is_image=True)
                messaging_api.push_message(
                    PushMessageRequest(to=user_id, messages=[TextMessage(text=description)])
                )
        except Exception as e:
            print(f"Error processing image: {e}")

    else:
        print(f"Received unsupported message type: {msg_type}")
```

**4. Resetting Chat History:**

Use `datetime` to track message timestamps and reset chat history after a certain interval (e.g., 10 minutes).

This enhanced code provides a solid foundation for a robust hydroponic system management server with improved security, error handling, and flexibility. You can further customize it to suit your specific project needs and integrate additional features for enhanced functionality. 
