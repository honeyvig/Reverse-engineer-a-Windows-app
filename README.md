# Reverse-engineer-a-Windows-app
The goal of the task is to reverse engineer a Windows app. The windows app communicates with a server through HTTP requests, with something that looks like binary encoded SQL queries. The goal is to reproduce calls done inside the app to read and write ressources. The output of the task should be a SDK written in python with functions to perform CRUD actions on the different ressources / tables. (I was able to inspect calls with Charles Proxy and inspect the exe with Ghidra, but need help to finish this). 
ChatGPT said:

Thanks for the detailed context—you're well on your way. Based on what you've described, you're trying to reverse-engineer an app that communicates with a remote server via a custom protocol over HTTP, likely using binary-encoded SQL (or a proprietary query format) to read/write data. Your goal is to fully understand and mimic that behavior in a Python SDK.

Here’s a step-by-step plan to help you finish this task efficiently:
✅ Phase 1: Understand the Protocol and Requests

Since you have Charles Proxy logs, do the following:
1. Capture and Analyze Requests

Look for:

    URL endpoints

    HTTP methods (POST, GET, etc.)

    Headers (especially any auth tokens, content-type, etc.)

    Binary payloads – extract and save them (use Charles "Save Body" or export to binary files)

2. Correlate with Ghidra Output

Use Ghidra to identify:

    Functions that send HTTP requests (look for WinHttpSendRequest, InternetOpenUrl, HttpSendRequest, etc.)

    Code that builds payloads before sending

    Look for string references like table names, SQL keywords, or fixed endpoint URLs

    Function names or structures that suggest CRUD operations (e.g. InsertUser, FetchLogs, etc.)

    Tip: Set breakpoints and monitor memory with something like x64dbg or API Monitor if needed.

✅ Phase 2: Reverse the Binary SQL/Query Format

If the binary data is an encoded query format (e.g. a custom or compressed SQL), try:
1. Compare Known Queries

If you perform a known action (e.g., fetch user 42) and capture the binary payload:

    Use that to reverse-map the payload format

    Compare similar queries with different params to locate where parameters go

2. Decode or Decompress

Check if the binary is:

    Compressed (e.g., gzip, zlib)

    Serialized (e.g., Protobuf, custom binary format)

    Encrypted (less likely unless there’s a handshake or certificate)

    Use tools like binwalk, CyberChef, or xxd/hexdump to inspect and manipulate.

✅ Phase 3: Build Python SDK

Once you understand the protocol, start building your SDK:
1. Set Up Communication Layer

import requests

class CustomAppClient:
    def __init__(self, base_url, auth_token=None):
        self.base_url = base_url
        self.session = requests.Session()
        if auth_token:
            self.session.headers.update({'Authorization': f'Bearer {auth_token}'})

2. Implement Binary Encoding/Decoding

Write functions to:

    Encode binary SQL/query formats

    Decode responses (often includes reading byte offsets, variable-length encoding, etc.)

Use Python's struct, bitstring, or construct libraries depending on the complexity.
✅ Phase 4: Implement CRUD Functions

Assuming you've mapped query patterns to actions:

class CustomAppClient:
    ...

    def get_resource(self, table, id):
        payload = encode_query(f"SELECT * FROM {table} WHERE id = {id}")
        response = self.session.post(self.base_url + '/query', data=payload)
        return decode_response(response.content)

    def create_resource(self, table, data_dict):
        payload = encode_query(f"INSERT INTO {table} ...")  # format as needed
        response = self.session.post(self.base_url + '/query', data=payload)
        return decode_response(response.content)

🔬 Want Help with Binary Analysis?

If you can share:

    A hex dump or file of a request

    Some known SQL-like queries with their binary payloads

    Any observations from Ghidra (e.g. struct layouts, constants, etc.)

Then I can help you reverse the encoding format directly and build Python parsers.

Would you like to start by sharing a captured binary payload and the action it represents?
🔍 What I Need from You:

    🔢 A Captured Binary Payload:

        Export the raw binary data of an HTTP request body captured by Charles Proxy (you can use .bin or .har if needed).

        Ideally include both the request and the server's response.

    🧾 The Action It Represents:

        Describe what this specific request is doing (e.g., "fetch user by ID = 42", or "insert a new log entry").

    💡 Known SQL-Like Equivalent (if possible):

        What would this binary payload translate to in plaintext SQL if known? (e.g., SELECT * FROM users WHERE id = 42)

    📐 Structs or Constants from Ghidra (optional but helpful):

        If you've identified any data structures, function names, or constants in Ghidra that seem related to the payload construction, please paste them or summarize.

✅ What You’ll Get From Me:

Once you provide that:

    I’ll reverse engineer the structure of the payload

    Provide a Python parser (decode_payload() function)

    Show how to build new binary payloads (encode_query()) for CRUD ops

    Package this into a reusable Python SDK

Ready when you are — just upload the files or paste the hex dump (or base64), and we’ll dig in!
