# RAG_Integration

Below is a comprehensive `README.md` file that provides step-by-step instructions for processing chunks from the Retrieval-Augmented Generator (RAG) API and enriching Jira user stories with contextual metadata.

---

# **Comprehensive Guide: Processing Chunks from RAG API and Enriching Jira User Stories**

This guide provides detailed instructions for integrating with the RAG API, processing chunked responses, and enriching Jira user stories with contextual metadata. By following these steps, developers can improve query precision and relevance in their RAG pipeline.

---

## **Table of Contents**
1. [Prerequisites](#prerequisites)
2. [Processing Chunks from RAG API](#processing-chunks-from-rag-api)
   - Importing Libraries
   - Setting API Credentials
   - Sending API Requests
   - Parsing Chunk Data
   - Error Handling
3. [Enriching Jira User Stories](#enriching-jira-user-stories)
   - Importing Libraries
   - Setting Jira API Token
   - Retrieving User Story Data
   - Adding Contextual Metadata
4. [Sample Test](#sample-test)
5. [Troubleshooting](#troubleshooting)
6. [API Documentation and Support Resources](#api-documentation-and-support-resources)

---

## **Prerequisites**

Before proceeding, ensure you have the following:

### **Required Libraries**
Install the necessary Python libraries using `pip`:
```bash
pip install requests pandas openpyxl sentence-transformers transformers faiss-cpu
```

### **RAG API Credentials**
- Obtain your RAG API credentials (e.g., API key or token).
- Store the credentials securely in an environment variable or configuration file.

### **Jira API Token**
- Generate a Jira API token from your Atlassian account.
- Store the token securely in an environment variable or configuration file.

---

## **Processing Chunks from RAG API**

### **Importing Libraries**
Start by importing the required libraries:
```python
import requests
import json
```

### **Setting API Credentials**
Set up your RAG API credentials:
```python
RAG_API_URL = "https://your-rag-api-endpoint.com/query"  # Replace with your RAG API endpoint
HEADERS = {
    "Content-Type": "application/json",
    "Authorization": "Bearer YOUR_RAG_API_TOKEN"  # Replace with your RAG API token
}
```

### **Sending API Requests**
Define a function to send queries to the RAG API and retrieve chunked responses:
```python
def query_rag_api(query, top_k=5):
    """
    Queries the RAG API and retrieves chunked responses.
    
    Args:
        query (str): The query to send to the RAG API.
        top_k (int): Number of top results to retrieve.
    
    Returns:
        list: A list of chunked responses from the RAG API.
    """
    payload = {
        "query": query,
        "top_k": top_k
    }
    response = requests.post(RAG_API_URL, headers=HEADERS, data=json.dumps(payload))
    if response.status_code == 200:
        return response.json().get("chunks", [])
    else:
        print(f"Error querying RAG API: {response.status_code}, {response.text}")
        return []
```

### **Parsing Chunk Data**
Process the chunked responses into a coherent format:
```python
def process_chunks(chunks):
    """
    Processes the chunked responses from the RAG API into coherent text.
    
    Args:
        chunks (list): A list of chunked responses from the RAG API.
    
    Returns:
        str: Combined and processed text from the chunks.
    """
    combined_text = "\n".join([chunk["text"] for chunk in chunks])
    return combined_text
```

### **Error Handling**
Handle errors gracefully by checking the API response status code and logging errors:
```python
try:
    chunks = query_rag_api("As an admin, I want to manage user permissions")
    if not chunks:
        raise ValueError("No chunks returned from RAG API.")
    processed_text = process_chunks(chunks)
    print(processed_text)
except Exception as e:
    print(f"An error occurred: {e}")
```

---

## **Enriching Jira User Stories**

### **Importing Libraries**
Import the required libraries for working with Jira data:
```python
import pandas as pd
from jira import JIRA
```

### **Setting Jira API Token**
Set up your Jira API token:
```python
JIRA_SERVER = "https://your-domain.atlassian.net"
JIRA_API_TOKEN = "YOUR_JIRA_API_TOKEN"  # Replace with your Jira API token
JIRA_EMAIL = "your-email@example.com"   # Replace with your Jira email

jira = JIRA(server=JIRA_SERVER, basic_auth=(JIRA_EMAIL, JIRA_API_TOKEN))
```

### **Retrieving User Story Data**
Fetch user story data from Jira:
```python
def fetch_jira_user_stories(project_key):
    """
    Fetches user stories from a Jira project.
    
    Args:
        project_key (str): The key of the Jira project.
    
    Returns:
        list: A list of user stories with relevant fields.
    """
    jql_query = f"project={project_key} AND issuetype=User Story"
    issues = jira.search_issues(jql_query, maxResults=100)
    user_stories = []
    for issue in issues:
        user_stories.append({
            "key": issue.key,
            "summary": issue.fields.summary,
            "description": issue.fields.description,
            "priority": issue.fields.priority.name,
            "labels": issue.fields.labels
        })
    return user_stories
```

### **Adding Contextual Metadata**
Enrich user stories with metadata such as tags, priority, and user roles:
```python
def enrich_user_story(user_story):
    """
    Enriches a Jira user story with contextual metadata.
    
    Args:
        user_story (dict): A dictionary containing user story details.
    
    Returns:
        dict: The enriched user story.
    """
    enriched_story = {
        "key": user_story["key"],
        "summary": user_story["summary"],
        "description": user_story["description"],
        "priority": user_story["priority"],
        "labels": user_story["labels"],
        "user_role": "admin" if "admin" in user_story["summary"].lower() else "customer",
        "keywords": user_story["summary"].split()
    }
    return enriched_story
```

---

## **Sample Test**

Here’s a complete example demonstrating API integration and user story enrichment:
```python
# Fetch user stories from Jira
project_key = "PROJ"
user_stories = fetch_jira_user_stories(project_key)

# Enrich user stories
enriched_stories = [enrich_user_story(story) for story in user_stories]

# Query RAG API with enriched context
for story in enriched_stories:
    query = f"{story['summary']} {story['description']}"
    chunks = query_rag_api(query)
    processed_text = process_chunks(chunks)
    print(f"Processed Text for {story['key']}:\n{processed_text}\n")
```

---

## **Troubleshooting**

### **Common Issues and Solutions**
1. **API Authentication Errors**:
   - Ensure your API tokens are valid and correctly formatted.
   - Check network connectivity and firewall settings.

2. **Empty Responses**:
   - Verify the query syntax and ensure the knowledge base contains relevant data.

3. **Chunk Parsing Errors**:
   - Validate the structure of the API response and handle missing fields.

### **API Documentation and Support Resources**
- **RAG API Documentation**: Refer to the official documentation for detailed usage instructions.
- **Jira API Documentation**: [Atlassian Developer Documentation](https://developer.atlassian.com/)
- **Support Channels**: Contact support via email or community forums for assistance.

---

## **Conclusion**

By following this guide, you can effectively integrate with the RAG API, process chunked responses, and enrich Jira user stories with contextual metadata. This approach ensures improved query precision and relevance in your RAG pipeline.
