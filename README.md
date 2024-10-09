# Krayon

Krayon is a command-line interface (CLI) tool designed to simplify the interaction and management of Large Language Models (LLMs). It provides a user-friendly way to communicate with LLMs, stream their output to the terminal, and leverage plugins for extended functionality.

[![Krayon](https://img.youtube.com/vi/_jhmjNOXzEo/0.jpg)](https://www.youtube.com/watch?v=_jhmjNOXzEo)

### Features:

- **Intuitive CLI for LLMs:** Offers a streamlined command-line interface for seamless interactions with various LLMs.
- **Streaming Output:** Streams LLM responses directly to your terminal, providing real-time feedback.
- **Pluggable Architecture:** Supports plugins to enhance Krayon's capabilities and integrate with other tools.
- **Built-in Plugin Server:** Includes a server for managing and registering plugins.
- **Plugin Invocation:** Easily invoke plugins within the CLI using the `@` syntax.
- **LLM Support:**  Works with popular LLMs like OpenAI and Anthropic.
- **Context Management:**  Provides commands like `/include`, `/save`, `/clear`, `/save_history`, and `/load_history` for managing context within a session.

## Getting Started

This section guides you on setting up and using Krayon locally.

### Prerequisites

Before using Krayon, ensure you have the following installed:

* Go Programming Language (Go 1.22.3 or later)

### Installation:

1. **Clone the repository:**
   ```bash
   git clone https://github.com/pyljain/krayon.git
   ```
2. **Navigate to the project directory:**
   ```bash
   cd krayon
   ```
3. **Build Krayon:**
   ```bash
   go build -o ky
   ```

   Make sure to include it in your local machine's PATH

### Usage:

1. **Initialize Krayon:**
   ```bash
   krayon init
   ```
   This command will prompt you to enter your API key, provider (OpenAI or Anthropic), desired model, and a name for your profile. You can create multiple profiles for different use cases or API keys.

2. **Run Krayon:**
   ```bash
   krayon [profile_name]
   ```
   Replace `[profile_name]` with the name of the profile you want to use. If you only have one profile, you can omit the profile name.

   Once Krayon is running, you can start interacting with the LLM by typing your questions or prompts. Krayon will stream the LLM's response to your terminal in real-time.

#### Commands:

- **`krayon init [--key <key>] [--provider <provider>] [--model <model>] [--name <name>] [--stream]`**:  Creates a new profile.
    - `--key`: Your API key for the LLM provider.
    - `--provider`: The LLM provider (default: `anthropic`). Supported values: `openai`, `anthropic`.
    - `--model`: The LLM model to use.
    - `--name`: The name for the profile.
    - `--stream`: Whether to stream the LLM's response (default: `false`).
- **`krayon [profile_name]`**: Runs Krayon with the specified profile.
- **`krayon plugins list`**: Lists available plugins.
- **`krayon plugins install <plugin-name> [--version <version>]`**: Installs a plugin.
    - `--version`: The plugin version to install (default: latest).
- **`krayon plugins server [--port <port>] [--driver <driver>] [--connection-string <connection_string>] [--storage-type <storage_type>] [--bucket <bucket>]`**: Starts the plugin server. 
    - `--port`: Specifies the port for the plugin server (default: 8000).
    - `--driver`: Database driver to use (default: `sqlite3`). Supported values: `postgres`, `sqlite3`.
    - `--connection-string`: Connection string for the database (default: `krayon_plugins.db`).
    - `--storage-type`: Type of storage to use for plugin binaries (default: `gcs`). Supported values: `gcs`.
    - `--bucket`: Name of the storage bucket to use for plugin binaries.
- **`krayon plugins register <plugin-name>`**: Registers a plugin (not yet implemented).

#### Slash Commands (within Krayon CLI):

- **`/include <file/directory/url>`**: Includes content from a file, directory, or URL into the context. If you don't specify a path, Krayon will present an interactive file picker.
- **`/save <file>`**: Saves the current AI response to a file.
- **`/save-history <file>`**: Saves the conversation history to a file.
- **`/load-history <file>`**: Loads conversation history from a file.
- **`/clear`**: Clears the current context.
- **`/exit`**: Exits the Krayon CLI.

### Plugin Management API

Krayon offers a plugin management API for registering and retrieving plugins. The server component of Krayon exposes a REST API that can be used to manage plugins.

#### Endpoints:

- **`POST /api/v1/plugins`**: Registers a new plugin.
    - Request Body:
        ```json
        {
          "name": "plugin_name",
          "description": "Plugin description",
          "version": "v0.0.1"
        }
        ```
        - Form Data:
            - `binary_mac`: Plugin binary for macOS.
            - `binary_windows`: Plugin binary for Windows.
            - `binary_linux`: Plugin binary for Linux.
- **`GET /api/v1/plugins`**: Fetches all plugins.
- **`GET /api/v1/plugins/{plugin_name}/versions`**: Fetches all versions of a specific plugin.
- **`GET /api/v1/plugins/{plugin_name}/versions/{version}/platforms/{platform}`**:  Downloads a plugin binary for a specific version and platform.

### Examples

#### Registering a Plugin:

```bash
curl -X POST \
    -H "content-type:multipart/form-data" \
    -F "name=readme" \
    -F "description=Improves upon your readme" \
    -F "version=v0.0.1" \
    -F binary_mac=@sample_plugins/readme/readme \
    -F binary_linux=@sample_plugins/readme/readme \
    -F binary_windows=@sample_plugins/readme/readme \
    http://localhost:8000/api/v1/plugins
```

#### Fetching All Plugins:

```bash
curl -v -X GET \
    http://localhost:8000/api/v1/plugins
```

### Plugin Development

Krayon plugins are developed as standalone executables that communicate with the main Krayon process over a Unix Domain Socket. Plugins can be written in any language that can communicate over a socket. Krayon provides a Go library that can be used to simplify plugin development.

#### Plugin API

Krayon exposes the following API endpoints for plugins:

- **`GET /api/v1/connect`**: Called by the plugin to establish a connection with the Krayon server and get the context for the request. The response body will contain a JSON object with the following structure:
    ```json
    {
      "question": "User's question or prompt",
      "history": [
        {
          "role": "user",
          "content": [
            {
              "text": "Previous user message",
              "type": "text"
            }
          ]
        },
        {
          "role": "assistant",
          "content": [
            {
              "text": "Previous AI response",
              "type": "text"
            }
          ]
        }
      ],
      "context": "Additional context from the user"
    }
    ```
- **`POST /api/v1/response`**: Used by the plugin to send responses back to Krayon. The request body should contain a JSON object with the following structure:
    ```json
    {
      "data": "Plugin's response to the user",
      "error": "Error message, if any"
    }
    ```
- **`POST /api/v1/llm`**: Used by the plugin to make requests to the LLM. The request body should contain a JSON object with the following structure:
    ```json
    {
      "messages": [
        {
          "role": "user",
          "content": [
            {
              "text": "User's message to the LLM",
              "type": "text"
            }
          ]
        }
      ]
    }
    ```
    The response body will contain a JSON object with the LLM's response, following the same structure as the `history` field in the `/api/v1/connect` response.

#### Plugin Example (Go)

```go
package main

import (
	"bytes"
	"context"
	_ "embed"
	"encoding/json"
	"flag"
	"fmt"
	"krayon/internal/llm"
	"krayon/internal/plugins"
	"log"
	"net"
	"net/http"
	"os"
	"time"
)

//go:embed README.md
var readmeContents string

func main() {
	f, err := os.OpenFile("readme.log", os.O_RDWR|os.O_CREATE|os.O_APPEND, 0666)
	if err != nil {
		log.Fatalf("error opening file: %v", err)
	}
	defer f.Close()

	log.SetOutput(f)
	socket := flag.String("socket", "", "Unix socket to listen on")
	flag.Parse()
	if socket == nil || *socket == "" {
		fmt.Println("No socket provided")
		os.Exit(-1)
		return
	}

	client := http.Client{
		Transport: &http.Transport{
			DialContext: func(_ context.Context, _, _ string) (net.Conn, error) {
				return net.Dial("unix", *socket)
			},
		},
	}

	time.Sleep(5 * time.Second)

	response, err := client.Get("http://unix/api/v1/connect")
	if err != nil {
		fmt.Println(err)
		os.Exit(-1)
		return
	}

	var info plugins.RequestInfo
	err = json.NewDecoder(response.Body).Decode(&info)
	if err != nil {
		fmt.Println(err)
		os.Exit(-1)
		return
	}

	go sendResponse(client, plugins.PluginResponse{
		Data:  "Sending request to generate a README\n",
		Error: "",
	})

	log.Printf("Constructing llmRequestBody")

	llmRequestBody := plugins.LlmRequest{
		Messages: []llm.Message{
			{
				Role: "system",
				Content: []llm.Content{
					{
						Text: `You are an experienced engineer who is excellent at tech documentation. 
										Could you review the provided README and generate an exemplar README from it for the project? `,
						ContentType: "text",
					},
				},
			},
			{
				Role: "user",
				Content: []llm.Content{
					{
						Text:        string(readmeContents),
						ContentType: "text",
					},
				},
			},
		},
	}

	log.Printf("llmRequestBody is %+v", llmRequestBody)

	buf := bytes.NewBuffer([]byte{})
	err = json.NewEncoder(buf).Encode(llmRequestBody)
	if err != nil {
		log.Printf("Error making a request to the llm %s", err)
		os.Exit(-1)
		return
	}

	log.Printf("Calling /llm")
	resp, err := client.Post("http://unix/api/v1/llm", "application/json", buf)
	log.Printf("Called /llm")
	if err != nil {
		log.Printf("Error in response from the /llm endpoint %s", err)
		os.Exit(-1)
		return
	}

	llmResponse := llm.Message{}
	err = json.NewDecoder(resp.Body).Decode(&llmResponse)
	log.Printf("LLM response %+v", &llmResponse)
	if err != nil {
		log.Printf("Error reading response from the LLM %s", err)
		os.Exit(-1)
		return
	}

	go sendResponse(client, plugins.PluginResponse{
		Data:  "\nWriting README",
		Error: "",
	})

	log.Printf("Writing README")
	err = os.WriteFile("README-AI.md", []byte(llmResponse.Content[0].Text), os.ModePerm)
	if err != nil {

		go sendResponse(client, plugins.PluginResponse{
			Data:  "",
			Error: err.Error(),
		})
		os.Exit(-1)
		return
	}

}

func sendResponse(client http.Client, response plugins.PluginResponse) {
	buf := bytes.NewBuffer([]byte{})
	err := json.NewEncoder(buf).Encode(response)
	if err != nil {
		log.Printf("Error sending response to /response %s", err)
		os.Exit(-1)
		return
	}

	_, err = client.Post("http://unix/api/v1/response", "application/json", buf)
	if err != nil {
		log.Printf("Error in response from the /response endpoint %s", err)
		os.Exit(-1)
		return
	}
}

```

### Contributing:

Contributions to Krayon are welcome! If you encounter any issues or have suggestions, please open an issue or submit a pull request.

### License:

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.




