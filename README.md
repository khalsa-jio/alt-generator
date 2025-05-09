Image - Alt Generator
========================

This module adds a new `AltText` field to the `File` DataObject and provides a simple Text field in Assets Admin while editing image to generate alt text using LLM clients which can be configured using yaml file.

https://github.com/user-attachments/assets/db0dcf62-0e49-45a7-9a16-a9b897892110

Requirements
------------

* Silverstripe framework ^5
* Silverstripe admin ^2
* Silverstripe asset-admin ^2
* KhalsaJio SilverStripe AI Nexus ^0.1

Installation
------------

```bash
composer require khalsa-jio/silverstripe-alt-generator
```

Configuration
-------------

You can configure the module using YAML file. Either create a new YAML file in your `app/_config` directory or add the configuration to an existing YAML file.

The configuration file should contain the following:

```yaml
---
Name: alt-generator
---

KhalsaJio\\AI\Nexus\LLMClient:
  default_client: KhalsaJio\AltGenerator\Client\OpenAI # or "KhalsaJio\AltGenerator\Client\Claude" - The default LLM client to use - required

  # Configurations for default client
SilverStripe\Core\Injector\Injector:
    KhalsaJio\AltGenerator\Client\OpenAI: # or "KhalsaJio\AltGenerator\Client\Claude"
        properties:
            ApiKey: '`OPENAI_API_KEY`' # can be set in .env file - required
            Model: 'gpt-4o-mini-2024-07-18' # default - optional
            CharacterLimit: 125 # default - optional, must be less than or equal to 200
            Prompt: '' # default can be found in the `AbstractLLClient` file under preparePrompt() method - optional

```

Custom Client
-------------

For those planning to use a different provider or who have their own client, you can create a your own custom LLM client by extending the `KhalsaJio\AI\Nexus\Provider\AbstractLLClient` class and adding `KhalsaJio\AltGenerator\Client\AltGeneratorTrait` trait. This gives you the freedom to implement your own alt text generation logic.

```php

namespace KhalsaJio\AltGenerator\Client;

use KhalsaJio\AI\Nexus\Provider\AbstractLLClient;
use KhalsaJio\AltGenerator\Client\AltGeneratorTrait;

class CustomLLClient extends AbstractLLClient
{
    use AltGeneratorTrait;
    /**
     * The API BASE_URL for your custom LLM client
     */
    protected $apiUrl = 'https://api.customllclient.com';

    /**
     * The default model for your custom LLM client
     */
    protected function getDefaultModel(): string
    {
        return 'custom-model';
    }

    /**
     * The name of your custom LLM client
     */
    public static function getClientName(): string
    {
        return 'CustomLLClient';
    }

     private function __construct()
    {
        parent::__construct();

        // This method is used to initialize the default prompt
        $this->initAltGenerator();
    }

    /**
     * This method is called to generate alt text for the image
     */
    public function generateAltText($base_64_image)
    {
        // This is example so you should change it according to your LLM client
        $requestBody = [
            'image' => $base_64_image,
            'model' => $this->getModel(),
            'prompt' => $this->preparePrompt(),
        ];

        return $this->chat($requestBody, 'API_ENDPOINT');
    }

    /**
     * This method is used to extract alt text from the response
     */
    protected function extractAltText($result): string
    {
        return $result['message'] ?? '';
    }

    /**
     * This method is used to extract usage data from the response
     */
    protected function extractUsageData($result): array
    {
        return $result['usage'] ?? [];
    }
}
```

Reporting Issues
----------------

Please [create an issue](https://github.com/khalsa-jio/silverstripe-alt-generator/issues) for any bugs you've found, or features you're missing.
