# Embedding Models

The following embedding wrappers are available:

| Embedding Model                        | Description                                                                                                                                                 |
|----------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| OpenAI                                 | OpenAI embeddings API.<br/>All models are supported - see OpenAI [docs](https://platform.openai.com/docs/guides/embeddings/embedding-models) for more info. |
| Cohere                                 | Cohere embeddings API.<br/>All models are supported - see Cohere [API docs](https://docs.cohere.com/reference/embed) for more info.                         |
| HuggingFace Inference API              | HuggingFace Inference API.<br/>All models supported by the API.                                                                                             |
| HuggingFace Embedding Inference Server | HuggingFace Embedding Inference Server.<br/>[Models supported](https://github.com/huggingface/text-embeddings-inference) by the inference server.           |
| Ollama                                 | Ollama embeddings API.<br/>All models are supported - see Ollama [models lib](https://ollama.com/library) for more info.                                    |
| Cloudflare Workers AI                  | Cloudflare Workers AI Embedding.<br/> For more info see [CF API Docs](https://developers.cloudflare.com/workers-ai/models/embedding/).                      |

## OpenAI

```go
package main

import (
	"context"
	"fmt"
	"os"

	openai "github.com/amikos-tech/chroma-go/openai"
)

func main() {
	ef, efErr := openai.NewOpenAIEmbeddingFunction(os.Getenv("OPENAI_API_KEY"), openai.WithModel(openai.TextEmbedding3Large))
	if efErr != nil {
		fmt.Printf("Error creating OpenAI embedding function: %s \n", efErr)
	}
	documents := []string{
		"Document 1 content here",
	}
	resp, reqErr := ef.EmbedDocuments(context.Background(), documents)
	if reqErr != nil {
		fmt.Printf("Error embedding documents: %s \n", reqErr)
	}
	fmt.Printf("Embedding response: %v \n", resp)
}
```

## Cohere

```go
package main

import (
	"context"
	"fmt"
	"os"

	cohere "github.com/amikos-tech/chroma-go/cohere"
)

func main() {
	ef := cohere.NewCohereEmbeddingFunction(os.Getenv("COHERE_API_KEY"))
	documents := []string{
		"Document 1 content here",
	}
	resp, reqErr := ef.EmbedDocuments(context.Background(), documents)
	if reqErr != nil {
		fmt.Printf("Error embedding documents: %s \n", reqErr)
	}
	fmt.Printf("Embedding response: %v \n", resp)
}
```

## HuggingFace Inference API

```go
package main

import (
	"context"
	"fmt"
	"os"

	huggingface "github.com/amikos-tech/chroma-go/hf"
)

func main() {
	ef := huggingface.NewHuggingFaceEmbeddingFunction(os.Getenv("HUGGINGFACE_API_KEY"), "sentence-transformers/all-MiniLM-L6-v2")
	documents := []string{
		"Document 1 content here",
	}
	resp, reqErr := ef.EmbedDocuments(context.Background(), documents)
	if reqErr != nil {
		fmt.Printf("Error embedding documents: %s \n", reqErr)
	}
	fmt.Printf("Embedding response: %v \n", resp)
}
```

## HuggingFace Embedding Inference Server

The embedding server allows you to run supported model locally on your machine with CPU and GPU inference. For more
information check the [HuggingFace Embedding Inference Server](https://github.com/huggingface/text-embeddings-inference)
repository.

```go
package main

import (
	"context"
	"fmt"

	huggingface "github.com/amikos-tech/chroma-go/hf"
)

func main() {
	ef, err := huggingface.NewHuggingFaceEmbeddingInferenceFunction("http://localhost:8001/embed") //set this to the URL of the HuggingFace Embedding Inference Server
	if err != nil {
		fmt.Printf("Error creating HuggingFace embedding function: %s \n", err)
	}
	documents := []string{
		"Document 1 content here",
	}
	resp, reqErr := ef.EmbedDocuments(context.Background(), documents)
	if reqErr != nil {
		fmt.Printf("Error embedding documents: %s \n", reqErr)
	}
	fmt.Printf("Embedding response: %v \n", resp)
}
```

## Ollama

!!! note "Assumptions"

    The below example assumes that you have an Ollama server running locally on `http://127.0.0.1:11434`. Use the following command to start the Ollama server:

    ```bash
    docker run -d -v ./ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
    docker exec -it ollama ollama run nomic-embed-text # press Ctrl+D to exit after model downloads successfully
    # test it
    curl http://localhost:11434/api/embeddings -d '{"model": "nomic-embed-text","prompt": "Here is an article about llamas..."}'
     ```

```go
package main

import (
	"context"
	"fmt"
	ollama "github.com/amikos-tech/chroma-go/ollama"
)

func main() {
	documents := []string{
		"Document 1 content here",
		"Document 2 content here",
	}
	// the `/api/embeddings` endpoint is automatically appended to the base URL
	ef, err := ollama.NewOllamaEmbeddingFunction(ollama.WithBaseURL("http://127.0.0.1:11434"), ollama.WithModel("nomic-embed-text"))
	if err != nil {
		fmt.Printf("Error creating Ollama embedding function: %s \n", err)
	}
	resp, err := ef.EmbedDocuments(context.Background(), documents)
	if err != nil {
		fmt.Printf("Error embedding documents: %s \n", err)
	}
	fmt.Printf("Embedding response: %v \n", resp)
}
```

## Cloudflare Workers AI

You will need to register for a Cloudflare account and create a API Token for Workers AI -
see [docs](https://developers.cloudflare.com/workers-ai/get-started/rest-api/#1-get-an-api-token) for more info.

```go
package main

import (
	"context"
	"fmt"
	cf "github.com/amikos-tech/chroma-go/embeddings/cloudflare"
)

func main() {
	documents := []string{
		"Document 1 content here",
		"Document 2 content here",
	}
	// Make sure that you have the `CF_API_TOKEN` and `CF_ACCOUNT_ID` set in your environment
	ef, err := cf.NewCloudflareEmbeddingFunction(cf.WithEnvAPIToken(), cf.WithEnvAccountID(), cf.WithDefaultModel("@cf/baai/bge-small-en-v1.5"))
	if err != nil {
		fmt.Printf("Error creating Cloudflare embedding function: %s \n", err)
	}
	resp, err := ef.EmbedDocuments(context.Background(), documents)
	if err != nil {
		fmt.Printf("Error embedding documents: %s \n", err)
	}
	fmt.Printf("Embedding response: %v \n", resp)
}
```
