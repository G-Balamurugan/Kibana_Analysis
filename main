import requests
import json
import re

# Given cURL command here 
curl_command = ''' 
'''

logInputFile = "/Users/balamurugan/Desktop/KibanaAnalysis/LogFile/sample.json"

# Function to parse cURL command and extract components
def parse_curl_command(curl_command):
    # Extract URL
    url_match = re.search(r"curl '(.*?)'", curl_command)
    url = url_match.group(1) if url_match else None

    # Extract headers
    headers_match = re.findall(r"-H '(.*?)'", curl_command)
    headers = dict(tuple(header.split(": ")) for header in headers_match)

    # Extract JSON body
    json_body_match = re.search(r"--data-raw '(.*?)'", curl_command)
    json_body = json.loads(json_body_match.group(1)) if json_body_match else None

    return url, headers, json_body

# Call the function to get URL, headers, and JSON body
url, headers, json_body = parse_curl_command(curl_command)

print("URL: ", url)
print("\nHEADERS:\n",headers)
print("\nJSON Body:\n",json.dumps(json_body, indent=1))

try:
    response = requests.post(url, headers=headers, json=json_body)
    if response.status_code == 200:
        try:
            json_data = response.json()
            json_object = json.dumps(json_data , indent=2)
            with open(logInputFile, "w") as outfile:
                outfile.write(json_object)
        except ValueError as e:
            print(f"Error parsing JSON: {e}")
    else:
        print(f"Error: {response.status_code} - {response.text}")
except Exception as e:
    print(f"An error occurred: {e}")
